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

我们再回到之前，`push`返回值为`false`，表示推入的数据已经达到缓冲的阈值，不建议继续推入，需要等缓冲中的数据被消费后才能继续推入。当然如果达到阈值后一直推送数据，消费不及的话，就会产生"背压/back-pressure"问题。

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

```javascript
const Writable = require('stream').Writable
const writeable = new Writable
writeable._write = (chunk, _, next) => {
	console.log(chunk.toString())
	next()
}
writeable.write('test1')
writeable.write('test2')
writeable.end('!')
writeable.on('finish', () => {
	console.log('写入完成')
})
```

类似`readable`必须实现一个`_read`方法，`writable`必须实现一个`_write`方法。可传三个参数，分别是写入内容、编码与回调方法。回调方法是必须，否则写入一次数据后则不能再继续写入数据。

写入数据最后，需要`end`，表示写入数据结束，再继续写入则失败。

同样，`writable`也有一个写入缓冲区——`highWaterMark`。用来控制写入缓冲区的大小。如果写入的数据大于缓冲区的阈值，则会返回`false`，建议不再继续写入。当缓冲区的数据消费完，会自动触发`drain`句柄，然后我们才可以继续写入。

### 背压模式/back-pressure

到此，咱们可以看下怎么解决背压问题。我们这边把`readable`与`writable`连接起来，通过`drain`来控制流的读取、写入。

```javascript
const fs = require('fs')
const Writable = require('stream').Writable
const writable = new Writable({
	highWaterMark: 2
})
let readable = fs.createReadStream('./package-lock.json', {
	highWaterMark: 4
})
writable._write = (chunk, _, next) => {
	next()
}
readable.on('data', (chunk) => {
	console.log(chunk.toString())
	if (!writable.write(chunk)) {
		readable.pause()
	}
})
writable.on('drain', () => {
	readable.resume()
})
```
可以看到，因为写入的缓冲区阈值较小，文件中的数据不是一下子全部输出的，在可读流暂停与开始间来回切换输出的。其实，这个也是`pipe`的核心代码。

#### objectMode

以上所有例子中，可读流与可写流都是传输的`Buffer`。我们可以把`objectMode`设置为`true`，就可以传除了`null`之外的其他js对象。

#### 使用实现

`es6`

```javascript
import { Writable } from stream
class myWritable extends Writable {
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
const Writable = require('stream').Writable
const util = require('util')

function myWritable() {
	Writable.call(this)
}

myWritable.prototype._read = () => {
}

util.inherits(myWritable, Writable)
```

---

## Duplex

`Duplex`是一个实现了`readable`与`writable`的流。比如`tcp socket`、`zlib`、`crypto`。我们可以理解为电话，能听能说话。

我们可以简单的实现一个既有`read`与`write`的双工流。

```javascript
const {Duplex} = require('stream')
const fs = require('fs')
let duplex = new Duplex

duplex._read = () => {
}

duplex._write = (chunk) => {
	console.log('write', chunk.toString())
}

duplex.on('data', (chunk) => {
	console.log(chunk.toString())
})

duplex.push('test')
duplex.write('write data')
```

我们日常所遇到的`a.pipe(b).pipe(a)`，则是一个`duplex`。

---

## Transform

`duplex`是一个既可读又可写的流，但是内部缓冲没有进行关联。进行关联的是`transform`。我们先可以实现一个先写后读的例子：

```javascript
const {Transform} = require('stream')
let transform = new Transform

transform._transform = (chunk, _, next) => {
	next(null, chunk.toString())
}

transform.pipe(process.stdout)

transform.write('a')
transform.end('b')
```

这里我们写入的字符串在`transfrom`方法处理后直接输出。也可以这样写：

```javascript
const {Transform} = require('stream')
let transform = new Transform

transform._transform = (chunk, _, next) => {
	console.log('-->', chunk.toString())
	transform.push(chunk.toString())
	next()
}

transform.pipe(process.stdout)

transform.write('a')
transform.write('b')
transform.write('c')
transform.write('d')
transform.end('f')
```

我们发现控制台输出是：

```bash
--> a
a--> b
b--> c
c--> d
d--> f
f%
```

可以明显看出，是一个写入-输出-写入-输出的顺序。那我们用`duplex`也试试呢？

```javascript
const {Duplex} = require('stream')
let duplex = new Duplex

duplex._read = () => {
}
duplex._write = (chunk, _, next) => {
	console.log('--->write', chunk.toString())
	duplex.push(chunk.toString())
	next()
}

duplex.pipe(process.stdout)
duplex.write('a')
duplex.write('b')
duplex.write('c')
duplex.write('d')
duplex.write('e')
duplex.end('f')
```

输出的为：

```bash
--->write a
--->write b
--->write c
--->write d
--->write e
--->write f
abcdef%
```

是全部写入后才进行的输出，证明了`transform`是共享内存的，`duplex`则没有。

`transfrom`可以监听`flush`来判断是否写入端数据已完成。



## 参考

[Stream文档](http://nodejs.cn/api/stream.html#stream_stream)

[美团stream三篇](https://tech.meituan.com/2016/07/08/stream-basics.html)

[对Node.js中 stream模块的学习积累和理解](https://github.com/zoubin/streamify-your-node-program)

[stream-handbook](https://github.com/jabez128/stream-handbook)

[Why Readable.push() return false every time Readable._read() is called
](https://stackoverflow.com/questions/38819196/why-readable-push-return-false-every-time-readable-read-is-called?rq=1)
