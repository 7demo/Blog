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

数据双向绑定是必须拦截`get`与`set`的。所以我们需要实现一个`observer`。

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
}
```

#### proxy

现在，我们都是通过`this.data.xx`的方式获取和设置数据的。我们可以通过一个`proxy`方法来实现`this.xx`读取数据。

```javascript
// observe.js

export const proxy = (tm, data) => {
	Object.keys(data).map(key => {
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
	// 初始化数据，拦截set与get操作
	observer(data)
	proxy(tm, data)
}

```




