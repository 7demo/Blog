# child_process

## `spawn`与`spawnSync`

### 语法：

```javascript
const {spawn} = require('child_process')
spawn('ls', ['-a'], {
	// ... options
	}）
subp.stdout.pipe(process.stdout)
```

+ cwd

	运行目录，默认当前目录。`cwd: '../'`

+ env

	运行环境变量。

	```javascript
		env: {
			environment: 'production'
		},
	```

+ arg0

	显式设置子进程arg[0]的值。`argv0: ''`

+ stdio

	用于在父子进程间建立通道，三个值分别对应`subprocess.stdin, subprocess.stdout, subprocess.stderr`， 也是`fd`对应索引`0/1/2`的值。其值可以分为：`pipe`/`ignore`/`inherit`。默认值是 `[pipe, pipe, pipe]`。

	- pipe

	`pipe`在父子进程间创立管道。

	```javascript
	// index.js
	let {spawn} = require('child_process')
	let subp = spawn('node', ['subprocess.js'], {
		stdio: ['pipe', 'pipe', 'pipe']
	})
	subp.stdout.pipe(process.stdout)
	// subprocess.js
	console.log('subprocess')
	```

	执行`node index.js`。此时输出的就是`stdout`。

	```javascript
	// index.js
	let {spawn} = require('child_process')
	let subp = spawn('node', ['subprocess.js'], {
		stdio: ['pipe', 'pipe', 'pipe']
	})
	subp.stderr.pipe(process.stderr)
	// subprocess.js
	test()
	```
	执行`node index.js`。此时输出的就是子进程的`stderr`。

	```javascript
	let {spawn} = require('child_process')
	let subp = spawn('node', ['subprocess.js'], {
		stdio: ['pipe', 'pipe', 'pipe']
	})

	// 因为异步，所以需要延迟往子进程中写入数据
	setTimeout(() => {
		subp.stdin.write('test')
		subp.stdin.end('test1')
	}, 100)

	// 监听子进程中的输出
	subp.stdout.pipe(process.stdout)
	```

	执行`node index.js`。此时输出的就是父进程中往子进程中写入都的数据。

	- stdio

	如果设置为`stdio`，则子进程的fd将会忽略，则上述例子中的`subp`则的`stdout/stdin/stderr`属性都为`null`而不是一个流对象。

	```bash
	ChildProcess {
	  _events: [Object: null prototype] {},
	  _eventsCount: 0,
	  _maxListeners: undefined,
	  _closesNeeded: 1,
	  _closesGot: 0,
	  connected: false,
	  signalCode: null,
	  exitCode: null,
	  killed: false,
	  spawnfile: 'node',
	  _handle:
	   Process { onexit: [Function], pid: 14407, [Symbol(owner)]: [Circular] },
	  spawnargs: [ 'node', 'subprocess.js' ],
	  pid: 14407,
	  stdin: null,
	  stdout: null,
	  stderr: null,
	  stdio: [ null, null, null ]
	}
	```

	- inherit

	表示子进程的`stderr/stdin/stdout`直接指向父进程`process`的`stderr/stdin/stdout`。

	```javascript
	// process
	let {spawn} = require('child_process')
	let subp = spawn('node', ['subprocess.js'], {
		stdio: 'inherit'
	})
	process.stdin.write('process')

	//subprocess.js
	console.log('subprocess')
	process.stdin.on('data', (chunk) => {
		console.log(chunk)
	})
	```

	运行，控制台直接输出子进程的打印值`process subprocess`。

	- stream

	传入流对象。`stdin`除了`process.stdin`都报错。待学习。`stdout`需要可写流触发`open`事件后（异步亦可）执行`swapn`或者是使用已经指定`fd`的可写流（因为不再有fd事件）。

	```javascript
	let {spawn} = require('child_process')
	let fs = require('fs')
	let out = fs.createWriteStream('out.log')
	// setTimeout(() => {
	// 	let subp = spawn('node', ['subprocess.js'], {
	// 		stdio: [process.stdin, out, 'pipe']
	// 	})
	// }, 0)
	out.once('open', () => {
		spawn('node', ['subprocess.js'], {
			stdio: [process.stdin, out, 'pipe']
		})
	})

	// 或者可写流直接指定fd
	let fd = fs.openSync('out.log', 'w') // 此时获取fd必须同步行为
	let out = fs.createWriteStream('out.log', {
		flags: 'a',
		fd: fd
	})
	spawn('node', ['subprocess.js'], {
		stdio: ['pipe', out, 'pipe']
	})
	```

	- 正整数。类似`stream`。

	```javascript
	let fd = fs.openSync('out.log', 'w') // 此时获取fd必须同步行为
	spawn('node', ['subprocess.js'], {
		stdio: ['pipe', out, 'pipe']
	})
	```

	- null/undefiled

	使用默认值。

	- ipc

	在父子进程间创建ipc通道。

	```javascript
	let {spawn} = require('child_process')
	spawn('node', ['subprocess.js'], {
		stdio: ['ipc']
	}).on('message', (data) => {
		console.log(data)
	})
	// subprocess.js
	console.log(1122)
	process.send('msg')
	```

	输出`msg`。也就是只能通过`send/on`来接受发送消息。

})

+ detached

	在window上，`detached：true`会父进程退出后子进程继续运行。子进程有自己的控制台窗口。

	非window上，`detached: true`会让子进程成为新的进程与会话的领导者。子进程在父进程退出后可以继续运行。

	默认情况下，父进程需要等待子进程退出。所以在`detached: true`为时，可以调用`subprocess.unref()`

	```javascript
	let {spawn} = require('child_process')
	let fs = require('fs')
	let out = fs.openSync('out.log', 'a')
	let prc = spawn('node', ['subprocess.js'], {
		detached: true,
		stdio: ['ignore', out, 'ignore']
	})
	prc.unref()

	// subprocess.js
	setInterval(() => {
		console.log(11111)
	}, 2000)
	```

	我们可以看到，控制台直接处于退出状态，并且`out.log`一直处于增加状态。

+ shell

	设置为`true`，则是在shell中运行。unix中默认是`/bin/sh`。

`spawn`创建异步进程，不阻塞事件循环。`spawnSync`创建同步进程，会阻塞事件循环。

```javascript
const {spawn} = require('child_process')
let ls = spawn('ls')
ls.stdout.pipe(process.stdout)
```


```
const {spawn} = require('child_process')
let ps = spawn('ps', ['-ef'])
let grep = spawn('grep', ['node'])

ps.stdout.pipe(grep.stdin)
grep.stdout.pipe(process.stdout)
```


`child_process`的其他方法都是基于`spawn`与`spawnSync`创建。

## `exec`与`execFile`

`exec`与`execFile`的区别主要是平台的区别。在类`unix`上，使用`execFile`,默认不衍生`shell`。在window平台，由于`.bat`与`cmd.exe`需要使用终端才能运行，需要使用`spawn`与`excel`。

###exec语法

第一个是执行命令，第二个参数是`options`，第三个参数是回调，接受`error/stdout/stderr`。我们主要看`options`。

+ encoding

	编码格式，默认是`utf-8`。

	```javascript
	exec('ls -a', {
		encoding: 'buffer'
	}, (err, ot) => {
		console.log(ot)
	})
	```

+ timeout

	超时时间，单位为好眠。如果值大于0时，子进程运行时间大于这个时间，则父进程会给子进程传递带`killSignal`属性的信号，结束进程。

	```javascript
	let {exec} = require('child_process')
	exec('node subprocess.js', {
		timeout: 1200
	}, (error, out) => {
		console.log(error, out)
	})
	// 子进程是一个setinterval函数。
	// 1200ms后输出：
	{ Error: Command failed: node subprocess.js
	    at ChildProcess.exithandler (child_process.js:294:12)
	    at ChildProcess.emit (events.js:182:13)
	    at maybeClose (internal/child_process.js:962:16)
	    at Process.ChildProcess._handle.onexit (internal/child_process.js:251:5)
	  killed: true,
	  code: null,
	  signal: 'SIGTERM',
	  cmd: 'node subprocess.js' }
	```

+ maxBuffer

	`stdout/stderr`上最大的字节数，默认200*1024。超过则停止。

	```javascript
	let {exec} = require('child_process')
	exec('ls -a', {
		maxBuffer: 1
	}, (error, out) => {
		console.log(error, out)
	})
	// 输出
	{ RangeError [ERR_CHILD_PROCESS_STDIO_MAXBUFFER]: stdout maxBuffer length exceeded
    at Socket.onChildStdout (child_process.js:348:14)
    at Socket.emit (events.js:182:13)
    at addChunk (_stream_readable.js:283:12)
    at readableAddChunk (_stream_readable.js:260:13)
    at Socket.Readable.push (_stream_readable.js:219:10)
    at Pipe.onStreamRead (internal/stream_base_commons.js:94:17) cmd: 'ls -a' } ''
	```

+ killSignal

	默认值为`SIGTERM`。其他值还可以为`SIGINT`、`SIGKILL`、`SIGTOP`。

	`SIGINT`可以由`ctrl+c`触发，只能结束前台进程。信号值是2.

	`SIGTERM`可以被忽略、处理与阻塞。信号值是15.

	`SIGKILL`强制终止程序。信号值是9.

	`SIGTOP`与`ctrl+d`触发。是挂起。信号值是20.

	```javascript
	exec('node subprocess.js', {
		killSignal: 'SIGINT',
		timeout: 1200
	}, (error, out, er) => {
		console.log(error, out, er)
	})
	//
	process.on('SIGINT', () => {
		process.exit()
	})
	```

```javascript
// window
spawn('cmd.exe', ['/c', 'my.bat'])
spawn('"my script.cmd"', ['a', 'b'], { shell: true })

exec('my.bat')
exec('"my script.cmd" a b')
```

```javascript
const {spawn, execFile} = require('child_process')
const {promisify} = require('util')
let execPromise = promisify(execFile)
execPromise('node', ['--version'], {
	encoding: 'buffer'
}).then((res) => {
	console.log(res)
})
```

## `fork`

`fork`是`spawn`的一个特例，专门用于生成进程。返回的`childProcess`会额外有一个通信通道，在父子进程间进行通信。根据之前`stdio`的配置，则其中须有一个值为`ipc`。每个进程都有自己的内存，会占用资源。

```javascript
let {fork} = require('child_process')
let child = fork('subprocess.js', {
})
// subprocess.js
setInterval(() => {
	console.log(11111)
}, 1000)
```

执行后，子进程不断输出内容。但是如果父进程crash，子进程则会一直输出成为一个孤儿进程。

```javascript
let {fork} = require('child_process')
let child = fork('subprocess.js', {
})
child.stdout.pipe(process.stdout) // 会报错。
// subprocess.js
setInterval(() => {
	console.log(11111)
}, 1000)
```


### 语法

	+ slient

	默认为false。表示继承父进程的`stdin/stdout/stderr`。为`true`时，表示`stdin/stdout/stderr`会输送到父进程。

	```javascript
	let {fork} = require('child_process')
	let child = fork('subprocess.js', {
		silent: true
	})
	child.stdout.pipe(process.stdout)
	```

	+ execArgv

	默认值`process.execArgv`。可执行文件的字符串参数列表。

	+ execPath

	用于创建子进程的可执行文件。

## child_process 事件

+ close

子进程`stdio`关闭时触发。与`exit`区别是，多进程可共享同一`stdio`。

+ disconnect

由`subprocess.disconnect()`与`process.connect()`。端口后`process.connected`的值为`false`。

+ error

可以由”无法衍生、无法杀死、向子进程发送消息失败“会触发。触发`error`后，`exit`可能不再触发。

+ subprocess.channel

IPC通道的引用。

+ subprocess.killed

是否接收到`kill`信号，但是不一定表示进程已结束。

+ subprocess.send

父子进程间创立通信通道时，发送消息使用。消息内容不能为：`{cmd: NODE_xxx}`，因为是node核心代码使用。发送消息失败会返回`false`。