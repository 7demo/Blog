# Vue源码学习

> 参考源码自我实现一个`Tue`。参考`vue`版本为`2.6.8`。

## 目录结构

> 我们也参考`vue`，使用`rollup`进行构建我们的程序。

```
├── dist # 编译后的文件
│   └── tue.js
├── example # 例子文件
├── package.json
├── scripts # rollup配置文件
│   └── config.js
├── src # 源文件
│   └── index.js
```

## Tue

看`vue`的文档，可以知道`Tue`是构函数或者类。生成实例时，

```javascript
// index.js
class Tue {
	constructor(options) {
		// console.log(options, this)
	}
}
```

我们在`Tue`的原型链上挂载一个`_init`方法，创建`Tue`实例时直接调用。

```javascript
// index.js
import {initMixin} from './init.js'
// ...
initMixin(Tue)
```

#### init

扩展原型链，增加`_init`方法。

```javascript
export const initMixin = (Tue) => {
	Tue.prototype._init = function (options) {
	}
}
```

#### proxy

我们都是通过`this.data.xx`的方式获取和设置数据的。我们可以通过一个`proxy`方法来实现`this.xx`读取数据。

```javascript
// proxy.js

export const proxy = (tm, data) => {
	Object.keys(data).map( key => {
		Object.defineProperty(tm, key, {
			set(val) {
				tm.data[key] = val
			},
			get() {
				return tm.data[key]
			}
		})
	})
}

```

```javascript
// init.js

Tue.prototype._init = function (options) {
	const tm = this
	const data = tm.data = options.data || {}
	proxy(tm, data)
}

```

#### compile

我们拿到数据后，需要去捕获模板中的key，换成对应的数据。

```javascript
// init.js

Tue.prototype._init = function (options) {
	const tm = this
	const data = tm.data = options.data || {}
	proxy(tm, data)
	compile(el, tm)
}

```

```javascript
// compile.js
export class Compile {
	constructor(el, tm) {
		this.tm = tm
		tm.$el = document.querySelector(el)
		// 节约性能
		let fragment = document.createDocumentFragment()
		let child = null
		while (child = tm.$el.firstChild) {
			fragment.appendChild(child)
		}
		let frag = this.replace(fragment)
		tm.$el.appendChild(frag)
	}
	replace(frag) {
		Array.from(frag.childNodes).map(node => {
			let txt = node.textContent
			let reg = /\{\{(.*?)\}\}/g
			// 文本
			if (node.nodeType === 3 && reg.test(txt)) {
				let val = this.tm
				let arr = RegExp.$1.split('.')
				arr.map(item => {
					val = val[item]
				})
				node.textContent = txt.replace(reg, val).trim()
			}
			// 节点
			if (node.nodeType === 1) {
				let attrs = node.attributes
				Array.from(attrs).map(attr => {
					let name = attr.name
					let exp = attr.value
					if (name.includes('v-')) {
						node.value = this.tm[exp]
					}
				})
			}
			// 递归遍历
			if (node.childNodes && node.childNodes.length) {
				this.replace(node)
			}
		})
		return frag
	}
}
```

至此，我们把页面给渲染出来了。但是如果要实现数据的动态绑定，还要继续改造。现在数据的绑定一般是通过`definePrototypy`拦截`get`与`set`的实现的。首先，我们要实现一个`observer`，拦截所有的数据的`get`与`set`。

#### obersver

```javascript
// observer.js

class Observer{
	constructor(data) {
		this.walk(data)
	}

	walk(data) {
		// 遍历所有的对象的所有key，进行拦截get与set
		Object.keys(data).map(key => {
			if (typeof data[key] === 'object') {
				// 如果是对象 继续遍历
				this.walk(data[key])
			}
			defineReactive(data, key, data[key])
		})
	}
}

// 具体拦截get 与 set
export const defineReactive = (obj, key, val) => {
	Object.defineProperty(obj, key, {
		set(newValue) {
			val = newValue
		},
		get() {
			return val
		}
	})
}

// 导出具体一个方法
export const observer = (data) => {
	return new Observer(data)
}
```

需要在`_init`中调用：

```javascript
Tue.prototype._init = function (options) {
	const tm = this
	const data = tm.data = options.data || {}
	// 初始化数据，拦截set与get操作
	observer(data)
	proxy(tm, data)
	compile(el, tm)
}
```

拦截`get`与`set`只是手段，实现数据观测并且触发编译才是目的。由于`data`中可能有无用数据，我们只需要监测有用数据，而哪些数据是有用的？在编译过程中使用到的就是有用的。

之前，我们要实现一个`dep`与`watcher`，用来收集监视的数据与监视数据并变化。

```javascript
// dep.js

export class Dep {
	constructor() {
		// 存放所有的监视器
		this.subs = []
	}
	addSub(sub) {
		// watcher
		this.subs.push(sub)
	}
	notify() {
		this.subs.map(sub => sub.update)
	}
}

Dep.target = null

```

```javascript
// watcher.js

export class Watcher {
	constructor(tm, exp, cb) {
		this.tm = tm
		this.exp = exp
		this.cb = cb
		// 初始化监视器时，会触发get方法
		this.get()
	}
	get() {
		// 当前监视器挂载dep上
		Dep.target = this
		// 主要是为了触发data的get方法,然后在get拦截器中把监视器放入数据依赖中
		let arr = this.exp.split('.')
		let var = this.tm
		arr.map(item => {
			val = val[item]
		})
		Dep.target = null
	}
	update() {
		let arr = this.exp.split('.')
		let var = this.tm
		arr.map(item => {
			val = val[teim]
		})
		// 拿到值 进行回调
		this.cb(val)
	}
}


```

刚才讲到，是在编译阶段开始收集依赖：

```javascript
// compile.js
replace(frag) {
	// ...
	node.textContent = txt.replace(reg, val).trim()
	// 新建立一个监视器
	new Watcher(this.tm, RegExp.$1, v => {
		node.textContent = txt.replace(reg, v).trim()
	})

	// ...
	// 新建立一个监视器
	node.value = this.tm[exp]
	new Watcher(this.tm, exp, v => {
		node.value = v
	})
}
```

为了把监视器放入依赖表中，则改动下`observer.js`：

```javascript
// observer.js
export const defineReactive = (obj, key, val) => {
	let dep = new Dep()
	Object.defineProperty(obj, key, {
		set(newValue) {
			val = newValue
			dep.notify()
		},
		get() {
			// 如果存在监视器则加入
			// 新建watcher会立即触发get方法
			Dep.target && dep.addSub(Dep.target)
			return val
		}
	})
}
```

现在只是单向数据绑定，如果实现双向绑定，则需要把`node`做下处理：

```javascript
// compile.js
node.addEventListener('input', e => {
	let nc = e.target.value
	this.tm[exp] = nc
})
```

#### 增加对数组的处理

现在把`data`中所有都收集`dep`中，数组也不例外，但是只支持数组的`set`并不支持`push/pop`等方法。我们要进行些改动。

在`observer`中需要对数组进行格外处理。

```javascript
// observer.js
class Observer{
	constructor(data) {
		// dep挂在observer的原因是 由于所有数据对象都已经被收集，所以触发数据更新也要同一dep
		this.dep = new Dep()
		this.walk(data)
	}

	walk(data) {
		Object.keys(data).map(key => {
			if (typeof data[key] === 'object') {
				this.walk(data[key])
			}
			// dep用于收集依赖与触发直接更改
			defineReactive(data, key, data[key], this.dep)
			if (Array.isArray(data[key])) {
				// 数组也用dep
				defineArrayReactive(data, key, this.dep)
			}
		})
	}
}

export const defineArrayReactive = (obj, key, dep) => {
	// 不影响array原原型链 创建一个原型对象
	let arrayProto = Array.prototype
	let arrayMethods = Object.create(arrayProto);

	// 遍历方法后，原型对象的方法纳入拦截
	[
		'push',
		'pop'
	].map(item => {
		Object.defineProperty(arrayMethods, item, {
			value: function(...arg) {
				// 使用原本原型链方法
				const original = arrayProto[item]
				let args = Array.from(arguments)
				original.apply(this, args)
				// 通知更新
				dep.notify()
			}
		})
	})
	// 把创建的纳入拦截的方法挂载到对象中的数组上。
	obj[key].__proto__ = arrayMethods
}
```

#### watch

`watch`是在初始化时，把所有的`watch`生成一个监视器`Watcher`，生成监视器过程，便与编译模板一样纳入了依赖管理，不过此处只是简单的触发，未做指定触发、去重等。

```javascript
// init.js
createWatch(tm, options.watch)

Tue.prototype.$watch = function(exp, cb) {
	new Watcher(this, exp, cb)
}
```

```javascript
// watcher.js
export const createWatch = (tm, watchs) => {
	Object.keys(watchs).map(key => {
		tm.$watch(key, watchs[key])
	})
}
```

为了能够在`watcher`的回调中拿到`odlval`与`newval`，需要在`watcher`中做下调整。

```javascript
// watcher.js
export class Watcher{
	constructor(tm, exp, cb) {
		//...
		this.value = this.get()
	},
	...
	update() {
		let arr = this.exp.split('.')
		let oldValue = this.value
		let val = this.tm
		arr.map(item => {
			val = val[item]
		})
		this.value = val
		this.cb(val, oldValue)
	}
```

