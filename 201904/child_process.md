# child_process

### `spawn`与`spawnSync`

`spawn`创建异步进程，不阻塞事件循环。`spawnSync`创建同步进程，会阻塞事件循环。

```javascript
const {spawn} = require('child_process')
let ls = spawn('ls')
ls.stdout.pipe(process.stdout)
```

`child_process`的其他方法都是基于`spawn`与`spawnSync`创建

### `exec`与`execFile`

`exec`与`execFile`的区别主要是平台的区别。在类`unix`上，使用`execFile`,默认不衍生`shell`。在window平台，由于`.bat`与`cmd.exe`需要使用终端才能运行，需要使用`spawn`与`excel`。

```javascript
// window
spawn('cmd.exe', ['/c', 'my.bat'])
spawn('"my script.cmd"', ['a', 'b'], { shell: true })

exec('my.bat')
exec('"my script.cmd" a b')
```