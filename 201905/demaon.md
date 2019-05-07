# 进程守护

> 读完`child_process.fork`与`cluster`，必然在想如何实现对自己程序的进程守护、多线程与监控重启。

实现进程的后台运行，关键的配置是`detached`。该值为`true`时，在会成为一个新会话领导者，父进程退出后依然存活。默认`false`。

首先看`child_process.fork`的实现：

```javascript
// parent.js
const {fork} = require('child_process')
let sub = fork('./subporcess.js', {
	detached: true
})
sub.unref()
//subprocess.js
const http = require('http')
http.createServer((req, res) => {
	res.writeHead(200)
	res.end('hello')
}).listen(3000)
console.log(`pid is ${process.pid}`)
```

运行`node parent.js`，然后`ctrl+c`退出。我们可以访问`localhost:3000`端口。查看node的进程也依然存在，说明我们的程序后台运行了。

那要子进程如果挂掉，如何重启呢？可以监听子进程是否失联。

```javascript
// parent.js
const {fork} = require('child_process')
let sub
let makeFork = () => {
	sub = fork('./subporcess.js', {
		detached: true
	})
	sub.unref()
	sub.on('disconnect', () => {
		console.log('退出disconnect')
		setTimeout(() => {
			makeFork()
		})
	})
}
makeFork()
```

如果同时多次`fork`，则会报端口已经占用。需要使用`cluster`。

```javascript
//parent
const cluster = require('cluster')
let makeFork = () => {
	let work = cluster.fork('./subporcess.js')
	work.on('disconnect', () => {
		setTimeout(() => {
			makeFork()
		}, 1000)
	})
}
if (cluster.isMaster) {
	for (let i = 0; i < 2; i ++) {
		makeFork()
	}
}
```

但是随着父进程的结束，子进程也会结束。得通过`spawn`变通下。

```javascript
//parent.js
const {spawn} = require('child_process')
spawn('node', ['subporcess.js'], {
	detached: true
})
setTimeout(() => {
	process.exit()
}, 100)
//subprocess.js
const http = require('http')
const cluster = require('cluster')
let makeFork = () => {
	let work = cluster.fork()
	work.on('disconnect', () => {
		setTimeout(() => {
			makeFork()
		}, 1000)
	})
}
if (cluster.isMaster) {
	for (let i = 0; i < 2; i ++) {
		makeFork()
	}
} else {
	http.createServer((req, res) => {
		res.writeHead(200)
		res.end('hello')
	}).listen(3000)
	console.log(`pid is ${process.pid}`)
}
```