# cluster

> 为了利用服务器多核的优势，可以使用cluster来创建多个进程。它本质是由`child_process`来创建的一组工作进程，父子进程间可以进行通信。工作进程获得任务由两种分配模式：

+ 循环法。除了window外的所有平台默认方法。

+ 主进程分发给感兴趣的进程。但是主动分发一般不是特别稳定：

  - `server.listen({fd: 7})`传递给工作进程的却是事件句柄而不是文件描述符。那工作进程直接使用句柄的情况下，就不再与主进程进行通信。造成父进程或者某一进程承担大部分工作，不能很好地实现负载。

  - `server.listen(0)`在`cluster`模式下，是伪随机。所有工作进程都是相同的随机端口。

  ```javascript
  	const cluster = require('cluster')
	const http = require('http')
	if (cluster.isMaster) {
	  cluster.fork()
	  cluster.fork()
	  cluster.fork()
	} else {
	  http.createServer((req, res) => {
	    res.writeHead(200)
	    res.end('hello world')
	  }).listen(0)
	  console.log(`工作进程 ${process.pid} 已启动`);
	}
  ```

  `cluster.fork`返回的是当前work对象。

## disconnect

```javascript
cluster.fork().on('disconnect', () => {
	console.log(1111, cluster.workers)
});
```

我们杀掉工作进程时，可以看到`cluster.workers`输出。

## exit

```javascript
let work = cluster.fork()
work.on('exit', (code, signal) => {
	console.log(1111, code, signal)
});
// 1111 null 'SIGTERM'
```

此外，还有其他`on`可以监听的句柄：`listening/online`

## message

我们了解到，父子进程间创建了一个通道，此通道就是靠`message`来进行通信。

子进程监听`message`:

```javascript
const cluster = require('cluster')
const http = require('http')
if (cluster.isMaster) {
  let work = cluster.fork()
  work.on('message', (obj) => {
    console.log(1111, obj)
  });
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('你好世界\n');
  }).listen(8000);
  process.send({
    test: 1
  })
  console.log(`工作进程 ${process.pid} 已启动`);
}
```

父进程监听`message`：

```javascript
const cluster = require('cluster')
const http = require('http')
if (cluster.isMaster) {
  let work = cluster.fork()
  work.send({
    test: 1
  })
} else {
  http.createServer((req, res) => {
    console.log(cluster.workers)
    res.writeHead(200);
    res.end('你好世界\n');
  }).listen(8000);
  process.on('message', (...arg) => {
    console.log(arg)
  })
  console.log(`工作进程 ${process.pid} 已启动`);
}
```

## worker.disconnect()

在工作进程内，可以关闭server，关闭ipc通道。在阻塞的情况下，可以格外调用`kill`。

```javascript
let work = cluster.fork()
work.disconnect()
let timeout = setTimeout(() => {
	work.kill()
}, 2000)
work.on('disconnect', () => {
	console.log('已关闭')
	clearTimeout('disconnect')
})
```

在`work.disconnect()/kill()`之后，`work.exitedAfterDisconnect`将返回true。在之前都是返回的`undefined`。

此外，`wrok.id`返回进程编号。`work.isConnect()`返回是否连接中。`isDead()`是否终止。

## worker.exit

任何一个进程关闭，都会触发`exit`事件。回调有三个参数`work/code/signal`。

```javascript
  let work = cluster.fork()
  work.disconnect()
  let timeout = setTimeout(() => {
    work.kill()
  }, 2000)
  work.on('disconnect', () => {
    console.log('已关闭')
    clearTimeout('disconnect')
  })
  work.on('exit', (...arg) => {
    console.log('已退出', arg)
  })
  // 已关闭
  // 已退出 [ 0, null ]
```

也看出，`disconnect`事件是在`exit`事件之前触发。

## 
