# nodejs调试指南

## cpu

### perf, FlameGraph

`perf`是`Linux Kernal`自带性能分析工具。能进行函数级与指令级的热点查找。centos系统直接`yum install -y perf`进行安装。

我们node运行`node --perf_basic_prof app.js`，会在`/tmp/`下生产一个`perf-#{id}.map`的文件。

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