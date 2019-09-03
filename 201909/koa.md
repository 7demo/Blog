# koa

> 我们根据`koa`的理解，尝试去自己实现一个`koa`。此处是参考`koa2`，本地`node`版本为`10.15.0`，直接使用`async/await`，不考虑`koa1`的情况。

## 起步

我们新建初始化`koa`的目录：

```
├── index.js
├── lib
└── package.json
```

其中`lib`是存放`koa`源码的文件夹，`index.js`是使用新`koa`的demo文件。

假如`lib`中`application.js`是主文件，根据文档可知，`application.js`中必定导出一个`class`，它的实例有一个`listen`方法，根据参数`port`来创建一个`http server`。那么我们的`application`开始代码就呼之欲出了：

```javascript
// lib/application
const debug = require('debug')('koa:application')
const http = require('http')
debug.enabled = true
class Koa {
    constructor() {
        debug('init koa')
    }
    listen(...arg) {
        debug(arg)
        let server = http.createServer()
        return server.listen(...arg)
    }
}

module.exports = Koa
```

```javascript
// index.js
const debug = require('debug')('koa:index')
const Koa = require('./lib/application')
const app = new Koa()
debug.enabled = true

app.listen(5000)
```
我们在程序根目录上，执行`node index.js`，就可以启动了一个`http`服务器。通过`telnet 127.0.0.1 5000`可以验证通过。当然我们也可以在`http.createServer`稍微做下改动:

```javascript
let server = http.createServer((request, response) => {
    response.writeHead(200, {
        'Content-type': 'text/html'
    })
    response.end('hello world')
})
```

重新启动，我们访问`http://127.0.0.1:5000`就看到`hello world`。但是，我们看文档，是使用`app.use((ctx, next) => {ctx.body = hello})`来实现网页访问内容的。所以，我们要首先弄明白`app.use`，即`koa`的中间件原理、洋葱结构。

## 中间件原理


