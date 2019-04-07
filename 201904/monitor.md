# 监控服务进程———自动重启

> 需要监控的进程为`sidekiq`，此服务为后台队列服务，过一段时间，此服务就会挂掉。

监控脚本`monit.sh`：

```bash
    #!/bin/bash
	echo $(pwd)
	flag=`ps aux|grep sidekiq | grep -v "grep"|wc -l`
	echo "$flag"
	if [ "$flag" -lt 1 ];then
	    cd /home/ruby/apps/carrera/current
	    echo $(pwd)
	    bundle exec sidekiq -e production
	    echo '程序已挂掉，重启中...'
	else
	    echo '程序运行中...'
	fi
```

给予执行权限`chmod +x monit.sh`

1, 获取`sidekiq`进程个数

2, 如果进程数少于1，则进入程序目录执行启动

我们编辑`/etc/crontab`，

```bash
   */1 *    * * *  root    /home/ruby/watch/monit.sh>/tmp/monit.log 2>&1 & # 每分钟执行一次，crontab执行日志就是/tmp/monit.log
```

我们发现原本直接执行没有问题的脚本，在定时器中发现各种`command not found`或者其他变量与环境问题。

很多人建议直接把原本命令的环境变量提前声明`PATH=...`，试过后不行。

有人建议，运行时执行`source /etc/profile`加载环境变量，试过后还是不妥。

查资料发现，<font color=#FF7F50>`crontab`执行脚本时是#无用户角色#的，也就无对应用户环境下的变量，既然无用户角色，那我们给他一个用户角色怎么样</font>？

```bash
    #!/bin/bash
	echo $(pwd)
	su user1
	source /etc/profile
	flag=`ps aux|grep sidekiq | grep -v "grep"|wc -l`
	echo "$flag"
	if [ "$flag" -lt 1 ];then
	    cd /home/ruby/apps/carrera/current
	    echo $(pwd)
	    bundle exec sidekiq -e production
	    echo '程序已挂掉，重启中...'
	else
	    echo '程序运行中...'
	fi
```

我们执行脚本时，先把切换成`user`然后再执行后面命令，完美避开变量问题。

不过问题又出现了，当重启进程时，会经常一下子重启4个或者5个。查资料后，原因应该是`crontab`的命令是异步执行的，可以使用`flock`来做一个锁。

试过后进程依然有多个，并且由于锁的存在执行一次后不再执行，这可不是所期待的效果。

由于是后台监控进程，所以多个进程暂无影响，所以脚本进行了稍微的改动，进程个数超过5个时，杀掉所有，来避免吃光内存。

```bash
echo $(pwd)
su user1
source /etc/profile
flag=`ps aux|grep sidekiq | grep -v "grep"|wc -l`
echo "$flag"
if [ "$flag" -lt 1 ];then
    cd /home/ruby/apps/carrera/current
    echo $(pwd)
    pid=`bundle exec sidekiq -e production`
    #pid=`nohup bundle exec sidekiq -e production \&;`
    #  cd /home/ruby/apps/carrera/current
    #  nohup bundle exec sidekiq -e production &
    #  bundle exec sidekiq -e production
    echo '程序已挂掉，重启'
    echo $pid
elif [ "$flag" -gt 5 ];then
    echo "$flag"
    echo 'shadiao'
    ps -ef | grep sidekiq | grep -v grep | awk '{print $2}' | xargs kill -9
else
    echo '程序运行中...'
fi
```