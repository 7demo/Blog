# 模块系统

### AMD

异步模块加载。分别定义模块与加载模块，加载`requre.js`，通过查找`data-main`来确定入口文件。

> 1. define是把所有生命函数变成一个module，存到全局waitings中

> 2. 查找入口文件，加载data-main，解析依赖

> 3. 依赖解析加载完，此时所有模块都会再全局中，触发回调函数。

```javaScript
/*
* r.js
*/
// 全局变量
const context = {
	topModule: '', // 顶层模块
	waitings: [], // 等待加载的模块
	loadeds: [], // 已加载的模块
	baseUrl: '',
	modules: [] // 所有模块
}

// 查找data-main来确定入口
let dataMainSrc = document.querySelector('[data-main]').getAttribute('data-main')

// 确定路径
let lastIndex = dataMainSrc.lastIndexOf('/')
if (lastIndex === -1) {
	context.baseUrl = './'
} else {
	context.baseUrl = dataMainSrc.substr(0, lastIndex)
}

// 删除元素
function removeEle(arr, ele) {
	let index = arr.indexOf(ele)
	if (index != -1) {
		arr.splice(index, 1)
	}
}

// 创建顶层节点 也就是data-main.js
let dataMainScript = document.createElement('script')
dataMainSrc.async = true
dataMainScript.src = dataMainSrc + '.js'
document.querySelector('head').appendChild(dataMainScript)
dataMainScript.onload = function() {
	removeEle(context.waitings, context.topModule)
	context.loadeds.push(context.topModule)
}

let tempModule = {}

function exec(moduleName) {
	let module = context.modules[moduleName]
	let deps = module.deps
	let args = []
	console.log('=====>>>>', moduleName)
	deps.forEach(function(dep) {
		exec(dep)
		args.push(context.modules[dep].returnValue)
	})
	module.args = args
	module.returnValue = context.modules[moduleName].factory.apply(context.modules[moduleName], args)
}

let require = function(deps, cb) {
	let moduleName = 'callback' + setTimeout(1)
	context.topModule = moduleName
	context.waitings.push(moduleName)

	// 生成模块配置
	context.modules[moduleName] = {
		moduleName: moduleName,
		deps: deps,
		factory: cb,
		args: [],
		returnValue: ''
	}

	// 遍历依赖
	deps.forEach(function(dep) {
		let depPath = context.baseUrl + dep + '.js'
		let script = document.createElement('script')
		script.setAttribute('data-module-name', dep)
		script.async = true
		script.setAttribute('src', depPath)
		context.waitings.push(dep)
		document.querySelector('head').appendChild(script)
		script.onload = function(e) {
			e = e || window.event
			let node = e.target
			let moduleName = node.getAttribute('data-module-name')
			tempModule.moduleName = moduleName
			context.modules[moduleName] = tempModule
			removeEle(context.waitings, moduleName)
			context.loadeds.push(moduleName)
			console.log('====', deps, depPath, context.waitings, context.loadeds)
			if (!context.waitings.length) {
				exec(context.topModule)
			}
		}
	})
}

window.require = require

let define = function(deps, factory) {
	tempModule = {
		deps: deps,
		args: [],
		returnValue: '',
		factory: factory
	}
	deps.forEach(function(dep) {
		let script = document.createElement('script')
		script.setAttribute('data-module-name', dep)
		script.async = true
		script.src = baseUrl.dep + '.js'
		document.querySelector('head').appendChild('script')
		script.onload = script
		context.waitings.push(dep)
	})
}

window.define = define
```

### CMD

同步加载

### commonjs

通过`require`加载，通过`module.exports`。

##### 加载顺序

0，如果有缓存直接拿缓存

1. 如果是内置模块，直接返回模块

2，如果模块名字包含有`./`和`../`，首先确定绝对路径，然后依次查找`x`/`x.js`/`x.json`/`x.node`，如果找不到，则把X看作一个目录：

    a, 根据`package.json`的`main`字段来确定

    b，找x/index.js

    c，找x/index.json

    d, 找x/index.json

3，如果没有路径，则首先查该目录下的`node_modules`，接着再查父目录


`exports`是`module.exports`的引用，`exports`相当于还是导出的`module.exports`。如果导出单个模块时，不能使用`exports=xx`


### es6

`script`可以增加属性`type=module`来表明是一个es6模块。

模块引入特性：

1，相同模块只能导入一次，如果多个模块引入相同模块，则只会再第一次import中执行

2，引入的模块是一个地址引用，会影响到其他

3，export default其实输出一个default的变量，后面不可以直接跟变声声明

```javaScript
export detault var a = 1 // 错误

var a = 1
export default a
```

##### export.default的问题


### cjs与es6互转


### import与require

1, require是执行时加载，import是编译时加载（效率高），这会造成import动作会提升到顶部

2，require加载的模块是对象，import只是输出代码片段，并且编译时执行可以做静态分析。

3，require是非严格模式，而import自动严格模式

4，commonjs是一个值得拷贝，而import是一个值得引用

5，es6的顶层this指向undefined

### webpack module

见[webpack](/201912/webpack.md)
