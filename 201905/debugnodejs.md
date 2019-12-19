# nodejs调试指南

## cpu

### perf, FlameGraph

#### 火焰图

`perf`是`Linux Kernal`自带性能分析工具。它是基于内核源码的一些`hook`——`Tracepoint`, 能进行函数级与指令级的热点查找。centos系统直接`yum install -y perf`进行安装。

我们node运行`node --perf_basic_prof app.js &`(加&会进入后台运行)，会在`/tmp/`下生产一个`perf-#{id}.map`的文件。

```bash
e3a46ee95b2 22 LazyCompile:~listenerCount events.js:489
e3a46ee974a f LazyCompile:~createServer http.js:33
e3a46ee9902 7b LazyCompile:~Server _http_server.js:264
e3a46ee9f82 16e LazyCompile:~Server net.js:1212
e3a46eeab72 235 LazyCompile:~Server.listen net.js:1440
e3a46eeb4d2 c4 LazyCompile:~normalizeArgs net.js:127
e3a46eeb85a 16 LazyCompile:~isPipeName net.js:82
e3a46eeba22 18 LazyCompile:~toNumber net.js:1261
e3a46eebbaa 4b LazyCompile:~isLegalPort internal/net.js:5
e3a46eebf92 b3 LazyCompile:~listenInCluster net.js:1398
e3a46eec51a 8 Script:~ cluster.js:1
...
```

每行信息都为3列，分别为16进制的符号地址、大小、符号名。注意参数`--perf_basic_prof`会使map内容增加，可使用`--perf_basic_prof_only_functions`，不过会导致火焰图不全。

`FlameGraph`是用于生成火焰图的工具。

我们首先获取运行的`node`进程——`ps -ef | grep node`。新开shell进行压测`$ ab -k -c 10 -n 2000 "http://localhost:3000/auth?username=admin&password=123456"`。同时，我们在另外一个shell根据node的进程采集信息——`perf record -F 99 -p 20316`。采集的信息生成`perf.data`文件。

执行`perf script > perf.stacks`生成`perf.stacks`。

执行`~/FlameGraph/stackcollapse-perf.pl --kernel < /tmp/perf.stacks | ~/FlameGraph/flamegraph.pl --color=js --hash> /tmp/flamegraph.svg`生成svg文件。

<img src="./images/flamegraph.svg?sanitize=true">

1，每一块代表一个函数

2，Y轴代表函数栈的深度，top代表占据cpu的函数。

3，x轴仅代表字母顺序。但是函数块的宽度代表占用cpu的时间。越宽代表使用cpu约长或者频繁。

4，多线程取样可能会超过运行时间。

上图可以看出`node::crypto::PBKDF2`占用时间比较多。如果我们在代码中把同步改为异步重新生成火焰图。

<img src="./images/flamegraph_async.svg?sanitize=true">

可以看到js代码中的宽度块所占宽度大大减少，同时多了一个`node:BackroundRunner`，是采用线程池异步执行来优化了性能。


#### 红蓝差火焰图

是基于优化前后的profile文件进行展示。如果优化后出现的次数多则为红色，否则为蓝色。

在代码修改前后分别执行：

```
perf record -F 99 -p <PID> -g -- sleep 30

perf script > perf_before.stacks
```

然后生成差分火焰图

```
~/FlameGraph/stackcollapse-perf.pl ~/perf_before.stacks > perf_before.folded
~/FlameGraph/stackcollapse-perf.pl ~/perf_after.stacks > perf_after.folded
~/FlameGraph/difffolded.pl perf_before.folded perf_after.folded | ~/FlameGraph/flamegraph.pl > flamegraph_diff.svg
```
<img src="./images/flamegraph_diff.svg?sanitize=true">


如果出现次数变少，则会是蓝色，就会造成一个问题就是如果优化后不再执行，那么就完全不没有蓝色显示。此时需要反差差分火焰图。

```
./FlameGraph/difffolded.pl perf_after.folded perf_before.folded | ./FlameGraph/flamegraph.pl --negate > flamegraph_diff2.svg
```

<img src="./images/flamegraph_diff2.svg?sanitize=true">

`flamegraph_diff`表示已修改前为基础，`flamegraph_diff2`表示已修改后为基准。`--negate`用于颠倒红蓝配色。

差分火焰图适合代码变化不大的情况。
