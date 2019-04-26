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

})

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


`child_process`的其他方法都是基于`spawn`与`spawnSync`创建

## `exec`与`execFile`

`exec`与`execFile`的区别主要是平台的区别。在类`unix`上，使用`execFile`,默认不衍生`shell`。在window平台，由于`.bat`与`cmd.exe`需要使用终端才能运行，需要使用`spawn`与`excel`。

```javascript
// window
spawn('cmd.exe', ['/c', 'my.bat'])
spawn('"my script.cmd"', ['a', 'b'], { shell: true })

exec('my.bat')
exec('"my script.cmd" a b')
```

`exec`与`execFile`接受一个回调，有三个参数`error`/`stdout`/`stderr`

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

`fork`是`spawn`的一个特例，专门用于生成进程。返回的`childProcess`会额外有一个通信通道，在父子进程间进行通信。

