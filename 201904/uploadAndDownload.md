# 文件上传下载的转发实现

> 一般，`node服务`都是作为一个中间层，渲染页面与转发请求。当请求是普通的`json`对象是则没有任何问题，但是需要我们把浏览器端上传的文件转发到服务端或者是把服务端的给的下载文件转发到前端呢？

> 这里，我们使用`koa`及相应的中间件来辅助实现。

## 文件上传转发

假如，我们浏览器是通过`formdata`发送请求的，比如：

```javascript
const file = document.querySelector('#file')
let formData = new FormData()
formData.append('name', 'test')
formData.append('file', file.files[0])
axios({
	url: '/upload',
	method: 'POST',
	headers: {
		'Content-type': 'multipart/form-data'
	},
	data: formData
}).then(res => {
	console.log(res)
})
```

我们node中间层server的代码如下：

```javascript
const Koa = require('koa')
const router = require('koa-router')()
const body = require('koa-body')
const view = require('koa-views')
const static = require('koa-static-cache')
const path = require('path')
const fs = require('fs')
const axios = require('axios')
const FormData = require('form-data')

const app = new Koa()
app.use(body({
	multipart: true
}))
app.use(view(__dirname + '/views'))

app.use(static(path.join(__dirname, 'public')))

router.get('/', async (ctx, next) => {
	await ctx.render('index')
})

router.post('/upload', async (ctx, next) => {
	console.log(ctx.request.files, ctx.request.body)
	ctx.body = {
		error: 0000
	}
})

app.use(router.routes())

app.listen(4001, '0.0.0.0')
console.log('listen on port 4001')
```

请求可以看到上传的文件信息与数据已经拿到。甚至可以通过以下代码把客户端的图片存在来（假设我们一直上传的都是一张jpg的图）：

```javascript
fs.createReadSteam('ctx.request.files[0].file').pipe(fs.createWriteStream('./1.jpg')) // file是浏览器端file的name
```

如果转发到后端，需要把文件流使用作为数据请求即可。这里用到了`from-data`:

```javascript
router.post('/upload', async (ctx, next) => {
	let formdata = new FormData()
	formdata.append('file', fs.createReadStream(ctx.request.files['file']['path']))
	formdata.append('name', 'test')
	await axios({
		url: 'http://127.0.0.1:4002/upload',
		method: 'post',
		headers: formdata.getHeaders(),
		data: formdata
	})
	ctx.body = {
		error: 0000
	}
})
```

其中，`formdata.getHeaders()`必须要设置。


## 文件下载转发

由于`koa`可以直接返回流的，所以在`node`请求服务端时，要求以流的形式返回，然后把这个流直接返回客户端即可。

在上面代码基础上实现一个`download`的路由。

```javascript
router.get('/download', async (ctx, next) => {
	let request = await axios({
		url: 'http://127.0.0.1:4002/download',
		method: 'get',
		responseType: 'stream'
	})
	ctx.attachment('./1.jpg')
	ctx.body = request.data
})
```

需要注意的时，浏览器会自动下载接受的流数据，所以这个地方需要通过`ctx.attachment('./1.jpg')`来指定名字与格式。
