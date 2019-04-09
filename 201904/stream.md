# Stream

> 正如[《stream-handbook》](https://github.com/substack/stream-handbook)中所认为，`stream`是要成为一个`nodejs`高手必须掌握的技能。


> `Stream`.正如其语义，一个有方向的流，在这条流上，我们可以随便取一段，对其就行干预处理。甚至，我们可以通过管道(`pipe`)让它与其他流连接起来。

---

## Readable/可读流

> Stream.Readable. 可读流，可以用于写入。常见的http请求、客户端响应、fs读取流，zlib流、crypto流、tcp socket、进程stdout与stderr、process.stdin。

首先看一个可读流的例子:

```JavaScript
const fs = require('fs')
let readStream = fs.createReadStream('./index.html')
readStream.on('data', (chunk) => {
	console.log('buffer chunk->', chunk)
	console.log('string chunk->', chunk.toString())
})
readStream.on('end', () => {
	console.log('end')
})
readStream.on('error', (err) => {
	console.log('err: ', err)
})
```

例子中，从`index.html`以流的形式读取数据，内存中只有一部分数据(缓冲区)，可以无视文件大小。对比`fs.readFile`，无论文件多大都是先读进内存，然后才处理，很容易爆掉内存。

### Readable流的使用

`Stream`是继承`EventEmitter`。在这儿，我们实现一个流，然后往流中推送数据输出到控制台。

```javascript
const Readable = require('stream').Readable
const readable = new Readable
readable.push('hello')
readable.push('\n')
readable.push('world')
readable.push(null)
readable.pipe(process.stdout)
```
首先，我们声明一个可读流。然后我们往这个流推送一些数据，然后在屏幕上输出。注意：`read.push(null)`是告诉流数据已推送完毕，是必须。

做js么，经常和异步同步打交道，这个地方很明显看出，流数据的推送是同步的，我们先把所有数据推送到流中，然后等待消费。流中的数据实际上是进入了缓冲区。我们肯定不希望这样，那如何实现异步呢？就是给可读流实现一个`_read`。

```javascript
const Readable = require('stream').Readable
const v = new Readable
readable._read = () => {
	console.log('_read 方法')
}
readable.pipe(process.stdout)

setTimeout(() => {
	readable.push('hello')
	readable.push('\n')
	readable.push('world')
	readable.push(null)
}, 2000)
```

这样，我们用`setTimeout`来模拟一个异步行为，发现2秒后数据才进行了输出。

`_read`基本上一个可读流必须实现的方法，流连接了底层数据。它是被系统底层调用的——流通过调用`_read`去请求数据，它调用后不会再被调用除非又`push`数据。

我们来证明下，每次都是底层调用`_read`。这里我们得首先明确，一旦一个可读流使用`pipe`或者`on('data')`就会不断的触发去读取数据(流动模式下)。

```javascript
const Readable = require('stream').Readable
const readable = new Readable
let c = 97 - 1
readable._read = () => {
	if (c >= 'z'.charCodeAt(0)) return readable.push(null);
	setTimeout(function () {
		readable.push(String.fromCharCode(++c));
	}, 100);
}
readable.pipe(process.stdout)
```

这里有两个地方需要说下：

1, 默认情况下会一直进行输出，但是这里在`read`进行了控制，大于`z`就不输出了。

2，正是`pipe`触发不断去读取数据，才能持续的调用`setTimeout`，否则的话只是一个同步推入数据。


`Readable`则是有一个缓冲区的，默认大小是`16`，通过`HighWaterMark`设置，单位是字节。每次可读流输出数据，都是先从从缓冲区读取，如果数据不够，则需要触发`read`去读取数据源。这里我们就明白了，`HighWaterMark`可以简单理解为一直在内存中的数据大小，这个值不能设置太小，否则读取太频繁，太大又会占用内存。

下面是一个例子：

```javascript
const Readable = require('stream').Readable
const readable = new Readable({
	highWaterMark: 10
})
readable.push('hello')
readable.push('\n')
readable.push('world')
readable.push(null)
readable.pipe(process.stdout)
```

这里，我们再看看`readable.push`的返回值。

```javascript
const Readable = require('stream').Readable
const readable = new Readable({
	highWaterMark: 10
})
let r1 = readable.push('hello')
let r2 = readable.push('\n')
let r3 = readable.push('world')
let r4 = readable.push(null)
console.log(r1, r2, r3, r4)
readable.pipe(process.stdout)
```

我们运行发现，输出`true, true, false, false`。这是什么意思？要知道这些，需要先了解流的几种状态。

#### 可读流的两种模式与三种状态

· 可读流有两种模式，流动模式——flowing与暂停模式——paused。

所有可读流开始都是暂停模式，在暂停模式中，必须显示的调用`read()`去读取数据。在流动模式下，会自动从底层读取数据。`on(data...)`、`pipe`与`resume`会自动把流从暂停模式切换到流动模式。可读流则通过`pause()`或者`unpipe()`来切到暂停模式。

· 三种状态，可以通过`readable.readableFlowing`来获取，分别是`null`、`false`、`true`对应可读流的初始化状态、暂停模式、流动模式。可以通过`readable.readableFlowing`来获取。

```javascript
const Readable = require('stream').Readable
const readable = new Readable
readable._read = () => {}
readable.pause()
readable.push('a')
setTimeout(() => {
	readable.resume()
	readable.push('b')
}, 1000)
readable.on('data', (chunk) => {
	console.log(chunk.toString())
})
```

我们看到，在运行1s后，先后输出`a/b`。说明暂停模式下，无法读取数据，直到`resume()`。同时也说明，暂停模式下，数据是先进入缓冲中，等待读取。

我们再回到之前，`push`返回值为`false`，表示推入的数据已经达到缓冲的阈值，不建议继续推入，需要等缓冲中的数据被消费后才能继续推入。

在暂停模式，我们需要监听`readable`主动去调用`read()`事件获取数据。

```javascript
const Readable = require('stream').Readable
const readable = new Readable
readable._read = () => {}
readable.pause()
readable.push('abcd')
readable.on('readable', () => {
	let data
	while (data = readable.read()) {
		console.log(data.toString())
	}
})
setTimeout(() => {
	readable.push('befg')
}, 2000)
```

`readable`表示有新数据或者是到了数据的尽头。这个地方，`read`方法可以传参`n`，来制定读取数据的大小字节数。如果传参为，则输出未`ab, cd, be, fg`。


#### 使用实现

`es6`

```javascript
import { Readable } from stream
class myReadable extends Readable {
	constructor(opts) {
		super()
		this.opts = opts
	}
	_read() {
	}
}
```

`js`

```javascript
const Readable = require('stream').Readable
const util = require('util')

function myReadable() {
	Readable.call(this)
}

myReadable.prototype._read = () => {
}

util.inherits(myReadable, Readable)
```

---

## Writable

> `Stream.Writable`，可写流。例子有：客户端的请求、服务端的响应、`fs`的写入流...子进程的`stdin`、`process.stdout`、`process.stderr`。



### 背压模式/back-pressure


## 参考

[Stream文档](http://nodejs.cn/api/stream.html#stream_stream)

[美团stream三篇](https://tech.meituan.com/2016/07/08/stream-basics.html)

[对Node.js中 stream模块的学习积累和理解](https://github.com/zoubin/streamify-your-node-program)

[stream-handbook](https://github.com/jabez128/stream-handbook)

[Why Readable.push() return false every time Readable._read() is called
](https://stackoverflow.com/questions/38819196/why-readable-push-return-false-every-time-readable-read-is-called?rq=1)
