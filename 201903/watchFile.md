# `fs.watch`与自动刷新

> 在前端开发提高生产力中，文件改动后实时的更新则属于非常有用的一种，无论在node开发中使用`nodemon`监视服务端文件变动，还是在前端页面开发中各种页面自动刷新都属于这类。因此在此对文件变动的监视与自动刷新的显示做些探究。

> 文件的监视则离不开`fs`的`watch`方法。

---

## `fs.watch`

#### `fs.watch`参数

`fs.watch`是`node`原生模块`fs`上的`watch`方法，可以监视文件或者文件夹的变动。

其可接受三个参数`(filename[, options] [, listener])`。

第一个参数是文件名或者文件夹名。

第二个参数可以是字符串或者对象，当是字符串是必须是`encoding`, 默认值为`utf-8`，代表监听文件的编码方式。

第三个参数是一个监听文件变动回调方法。它接受两个参数`(eventType, filename)`，分别为文件变动类型与变动文件的路径。其中`evnetType`的值可为`rename`与`change`。

现在来做些测试，首先创建文件夹文件如下：

```
|-- demo
	|-- src
		|-- file.js
	|-- index.js
```

```javasript // index.js
	const fs =  require('fs')

	fs.watch('./src/file.js', (eventType, filename) => {
		console.log(eventType, filename)
	})
```

此时，若在`src/file.js`中新增

```javascript
	const name = "test"
```
保存，然后运行`node index.js`可以在控制台看到如下输出：

```bash
	'change' 'file.js'
```

也就是说`file.js`做了`change`。不过我们发现`filename`只是文件的名字，不是一个相对或者绝对路径。我们试试把监控路径改成一个文件夹呢？

```javasript // index.js
	fs.watch('./src', {
		recursive: true // 监控子文件件的变化
	}, (eventType, filename) => {
		console.log(eventType, filename)
	})
```

当编辑`file.js`时，输出的`filename`依然只有`file.js`，并未带路由路径。

我们在`src`文件夹新增一个文件试试。

```
|-- demo
	|-- src
		|--childSrc
			|-- file.js
		|-- file.js
	|-- index.js
```

我们在控制台发现输出：

`rename childSrc`
`rename chhildSrc/file.js`

此刻编辑`childSrc/files`文件则有对应的文件夹输出，说明路径其实相对于监控的路径。
同时我们监听一个文件夹时，文件夹内部新增一个文件夹或者文件时时则认为`rename`。

此时，我们删除`childSrc/files`文件，控制台输出：

```bash
rename childSrc/file.js
```

删除文件或者文件夹时，触发的事件依然未`rename`。我们继续新增一个`childSrc/file.js`。控制台依然会输出`rename childSrc/file.js`。编辑则如常。


我们把`src`删除再新增一个`src/file.js`。发现删除与新增`src`时并未有回调提示，这是因为：

> 在 Linux 或 macOS 系统上， fs.watch() 解析路径到索引节点并监视该索引节点。 如果删除并重新创建监视的路径，则会为其分配一个新的索引节点。 监视器会因删除而触发事件，但会继续监视原始的索引节点。 不会因新建索引节点而触发事件。

除此之外，`watch`第二个参数为对象时，`persistent`默认为`true`表示持续监听，如果为`false`表示文件变动监听最后一个变动回调时退出。


#### `fa.watch`返回值

`fs.watch`返回值为`fs.FSWatch`，可以绑定`change`事件，其监听函数参数可以为一个数组，为事件类型与文件名称。

```javascript
const fs =  require('fs')
let fsStatus = fs.watch('./src', {
	recursive: true
}, (eventType, filename) => {
	console.log(eventType, filename)
})
fsStatus.on('change', (...arg) => {
	console.log('arg->', arg)
})
```
改动时，输出未`arg-> [ 'change', '1.js' ]`

------

### `fs.watchFile`


`fs.watchFile`参数与`fs.watch`类似，不同的是：

· 第一个参数只能是`filename`，不能是文件夹
· 第二个参数为对象，可省略。其属性值除了`persistent`，还有一个是`interval`，表示轮询文件的频率，毫秒为单位。
· 第三个参数是一个回调函数，两个参数分别代表当前与之前文件的状态。

```javascript
fs.watchFile('./src/test.js', (cur, pre) => {
	console.log(cur, pre)
})
```

编辑后输出为：

```bash
Stats {
  dev: 16777220,
  mode: 33188,
  nlink: 1,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4194304,
  ino: 24058197,
  size: 4,
  blocks: 8,
  atimeMs: 1552316136702.658,
  mtimeMs: 1552316132420.4565,
  ctimeMs: 1552316132420.4565,
  birthtimeMs: 1552212315942.2485,
  atime: 2019-03-11T14:55:36.703Z,
  mtime: 2019-03-11T14:55:32.420Z,
  ctime: 2019-03-11T14:55:32.420Z,
  birthtime: 2019-03-10T10:05:15.942Z }

  Stats {
  dev: 16777220,
  mode: 33188,
  nlink: 1,
  uid: 501,
  gid: 20,
  rdev: 0,
  blksize: 4194304,
  ino: 24058197,
  size: 0,
  blocks: 0,
  atimeMs: 1552316130774.0374,
  mtimeMs: 1552212320475.3477,
  ctimeMs: 1552212320475.3477,
  birthtimeMs: 1552212315942.2485,
  atime: 2019-03-11T14:55:30.774Z,
  mtime: 2019-03-10T10:05:20.475Z,
  ctime: 2019-03-10T10:05:20.475Z,
  birthtime: 2019-03-10T10:05:15.942Z }
```

我们发现，主要是文件大小与修改时间发生了改变。

同样，`fs.watchFile`返回值同`fs.watch`。

## 自动刷新的实现

记得刚开始从事这一行的时候，当时流行的是`F5`自动刷新——开发页面时，每次按保存，都会实现浏览器的自动刷新。这相对于每次手动刷新浏览器的的确确的提高了生产效率，尤其是对多屏幕的同学。

此刻，基于我们刚刚对`fs`的学习，是不是可以自我实现自动刷新功能呢？答案是肯定的，不过需要`websocket`的帮助。

首先安装`koa`、`ws`用于创建服务器与`websocket`服务器。创建目录结构：

```
|-- autoF
	|-- index.html
	|-- server.js
	|-- websocket.js
```
`index.html`文件，主要是html页面，用于展示、连接`websocket`服务器。
```html
	<!DOCTYPE html>
	<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title>Document</title>
	</head>
	<body>
		<h4>auto reload</h4>
		<script>
			// 连接ws 服务器
			let ws = new WebSocket(`ws://localhost:8081`);
			ws.onopen = function(arg) {
				console.log('已连接：', arg)
			}
			// 监听服务端发送的消息，实现刷新
			ws.onmessage = function(msg) {
				console.log('监听到消息', msg)
				if (msg.data === 'reload') {
					location.reload()
				}
			}
			ws.onerror = function(err) {
				console.log('err:', err)
			}
			ws.onclose = function (msg) {
				console.log('断开连接: ', msg)
			}
		</script>
	</body>
	</html>
```

`server.js`，是一个基于`koa`实现的`web server`。主要用于渲染页面、引入`ws`监控页面。
```javascript
const Koa = require('koa')
const ws = require('./websocket')
const app = new Koa()
const fs = require('fs')

app.use(ws())

// 所有请求都会返回index.html内容
app.use(ctx => {
	let file = fs.readFileSync('./index.html', 'utf-8')
	ctx.body = file
})

app.listen(8080, '0.0.0.0')

console.log('app server is running 8080!')
```

`websocket.js`，主要是实现`ws`服务与`watch`页面。

```javascript
const WebSocket = require('ws')
const fs =  require('fs')
// 创建ws server
const server = new WebSocket.Server({
  port: 8081,
  perMessageDeflate: {
    zlibDeflateOptions: {
      // See zlib defaults.
      chunkSize: 1024,
      memLevel: 7,
      level: 3
    },
    zlibInflateOptions: {
      chunkSize: 10 * 1024
    },
    clientNoContextTakeover: true, // Defaults to negotiated value.
    serverNoContextTakeover: true, // Defaults to negotiated value.
    serverMaxWindowBits: 10, // Defaults to negotiated value.
    concurrencyLimit: 10, // Limits zlib concurrency for perf.
    threshold: 1024 // Size (in bytes) below which messages
  }
})

class Server {
  constructor(ctx, next) {
    this.wsServer = server
    this.ctx = ctx
    // 如果连接成功，则执行connection函数
    this.wsServer.on('connection', this.onConnection.bind(this))
    next()
  }
  onConnection(ws) {
    console.log('ws server is connected!')
    // 监听文件改变
    fs.watch('./index.html', {
     recursive: true
    }, (eventType, filename) => {
      // 告诉浏览器 要刷新了~
      this.ctx.ws.send('reload')
    })
  }
}

module.exports = () => {
  return function(ctx, next) {
    return new Server(ctx, next)
  }
}
```

接下来，运行`node server.js`，在浏览器访问`http://localhost:8080/`。

如果我们此时编辑`index.html`页面，就会发现浏览器实现了自动刷新。

当然，以上只是个`demo`，仅仅展现了实现自动刷新的原理，我们在文件注入部分还可以自动注入页面中`ws`的连接代码，还得解决下`ws`刷新可能存在的内存泄漏问题。

基于以上学习，我们在`koa-views`代码的基础上，实现了一个可以实时刷新的(`koa-views`)[https://github.com/7demo/koa-liveload-views]

## 基于`EventSource`的实时刷新

上面代码实现的实时刷新中，浏览器页面与`server`的通信主要是基于`websocket`。`websocket`是一个浏览器与服务器的双向通信，这个对服务器的要求比较高，还需要额外配置`ws`的服务器。哪有没有更简单些的办法呢。答案肯定是有的，那就是`EventSouce`——`SSE`(server-send-event)。它与`websocket`区别的是——单向通信，在股票数据更新、新闻推送中更适用。因为这些场景不需要客户端推送信息到服务端。

现在我们开看看`EeventSource`的实现。

```server.js
const Koa = require('koa')
const app = new Koa()
const Router = require('koa-router')
const router = new Router()
const Readable = require('stream').Readable
const fs = require('fs')


function RR() {
	Readable.call(this, arguments)
}
RR.prototype = new Readable()
RR.prototype._read = function(data) {
}

function sse(stream, event, data) {
	return stream.push(`event:${event}\ndata:${JSON.stringify(data)}\n\n`)
}

router.get('/', (ctx, next) => {
	let file = fs.readFileSync('./index.html', 'utf-8')
	ctx.body = file
})

router.get('/sse', (ctx, next) => {
	var stream = new RR()
	ctx.set({
		'Content-type': 'text/event-stream',
		'Cache-Control': 'no-cache',
		'Connection': 'keep-alive'
	})
	ctx.body = stream
	fs.watch('./index.html', () => {
		sse(stream, 'test', {a: 2})
	})
})

app.use(router.routes())
	.use(router.allowedMethods())

app.listen(8080, '0.0.0.0')

console.log('app server is running 8080!')

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<h4>auto reload test</h4>
	<script>
		var sse = new EventSource('http://localhost:8080/sse')
		sse.addEventListener('test', (e) => {
			console.log('evnet', e)
			location.reload()
		})
		sse.addEventListener('error', (e) => {
			console.log('error', e)
		})
	</script>
</body>
</html>
```

以上就是代码，我们在控制台运行`node server.js`后，在浏览器访问`http://localhost:8080/`。如果我们编辑`index.html`，则会发现浏览器进行了刷新。

其实，这个`EventSource`也是`webpack`插件热更新实现的原理。这个与`eventSource`、`stream`则后续进行分析。



