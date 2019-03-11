# `fs.watch`与自动刷新的实现

> 无论在node开发中使用`nodemon`监视服务端文件变动，还是在前端页面开发中各种热更新（其实server中文件的变动也属于热更新）都属于对文件的监听。因此在此对其实现做些探查。

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
