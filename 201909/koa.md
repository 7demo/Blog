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

重新启动，我们访问`http://127.0.0.1:5000`就看到`hello world`。但是，我们看文档，是使用`app.use((ctx, next) => {ctx.body = hello})`来实现网页访问内容的。所以，我们要首先弄明白`app.use`，即`koa`的[中间件原理、洋葱结构](/201909/koa-compose.md)。

## 回调与中间件

通过`use`来使用中间件，所以`Koa`是通过`use`来存中间件的。

```javascript
constructor() {
    debug('init koa')
    this.middleware = []
}
use(fn) {
    this.middleware.push(fn)
    return this
}
...
```

此处`return this`是为了链式调用。

已经存的中间件怎么使用呢？可以在`http.creteServer`的回调函数中处理。

```javascript
callback() {
    let fn = compose(this.middleware)
    let hq = (request, response) => {
        fn(request, response)
    }
    return hq
}
listen(...arg) {
    debug(arg)
    let server = http.createServer(this.callback())
    return server.listen(...arg)
}
```

虽然在回调中开始处理中间件了，但是我们中间件的参数都是`(ctx, next)`。我们需要还有一个方法把`req/res`变成`ctx`的方法。首先得看下，`ctx`是怎么样的一个对象。

```
{ request:
   { method: 'GET',
     url: '/favicon.ico',
     header:
      { host: '192.168.10.243:8000',
        connection: 'keep-alive',
        pragma: 'no-cache',
        'cache-control': 'no-cache',
        'user-agent':
         'Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1',
        accept: 'image/webp,image/apng,image/*,*/*;q=0.8',
        referer: 'http://192.168.10.243:8000/',
        'accept-encoding': 'gzip, deflate',
        'accept-language': 'zh-CN,zh;q=0.9,en;q=0.8',
        cookie:
         'Hm_lvt_92363215421a5b2ffe0e08527f05de39=1564455576; Hm_lpvt_92363215421a5b2ffe0e08527f05de39=1564455576; Hm_lvt_c658b1c0eb5d6afef2ab2025561215ef=1567475937; JSESSIONID=abcZauh88r6vA_P6q20Zw; Hm_lpvt_c658b1c0eb5d6afef2ab2025561215ef=1567475968; currentRoute=%7B%22path%22%3A%22/transaction/transferlineup%22%2C%22params%22%3A%7B%7D%2C%22query%22%3A%7B%7D%7D; jumpAddress=http%3A//192.168.10.243%3A2300/transaction/transferlineup%3Fpopup%3Dtrue' } },
  response:
   { status: 404,
     message: 'Not Found',
     header: [Object: null prototype] {} },
  app: { subdomainOffset: 2, proxy: false, env: 'development' },
  originalUrl: '/favicon.ico',
  req: '<original node req>',
  res: '<original node res>',
  socket: '<original node socket>' }
```

`ctx`拥有`request/response/app/originlUrl/req/res/socket`属性。

我们新建`context.js`/`request.js`/`response.js`, 都只导出一个空对象。之所以建立对应的三个文件，是为了后面扩展原型链。

```javascript
// context.js
module.exports = {

}
```

```javascript
// request.js
module.exports = {
    
}
```

```javascript
// response.js
module.exports = {
    
}
```

然后在`application.js`中引入三个模块。

```javascript
//...
const context = require('./context')
const request = require('./request')
const response = require('./response')
//...
```

对`this.callback`进行改造：

```javascript
//...
callback() {
    let fn = compose(this.middleware)
    let hq = (request, response) => {
        let ctx = this.createContext(request, response)
        return this.handleRequest(ctx, fn)
    }
    return hq
}
```

主要是根据`request/response`创建`ctx`，拿到中间件继续进一步处理`this.handleRequest`。此处的第二个参数就是`next`。

```javascript
createContext(request, response) {

}
```