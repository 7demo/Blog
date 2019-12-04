# Proxy

### Proxy

> 在目标对象前设置一层拦截，拦截外界的访问，进行过滤与改写。

`Proxy`的语法是`obj = new Proxy(target, handler)`, 必须操作实例obj，如果handler是一个空对象，则是表示不做任何操作。需要注意的是，如果看起拦截，则必须在handler中对应的拦截中执行设置、读取、删除等操作。

```javaScript
var obj = new Proxy({}, {
    get(target, key, receiver){
        console.log('get: ', target, key, receiver) // receiver表示obj对象
        return target[key] + 2
    },
    set(target, key, value, receiver) {
        console.log('set: ', target, key, value, receiver)
        return target[key] = value - 1
    }
})
obj.count = 1 // set: {} count 1 Proxy {}
obj.count // 2 get: {count: 0} count Proxy {count: 0}
```

可以看到可以像`Object.definePrototype`一样，来拦截get与set操作。

`Proxy`不但可以拦截`set/get`，还可以有:

如果属性是一个不可配置与不可写，则不能通过proxy修改，set时，则修改无效。

```javaScript
const target = Object.defineProperties({}, {
    foo: {
        value: 1,
        writable: false,
        configurable: false
    }
})
const obj = new Proxy(target, {
    get(target, key){
        return 2
    }
})
```

`has(taget, key)`拦截`key in proxy`。如果原对象时禁止扩展，则会报错。`Object.preventExtension`

`deleteProperty(target, key)`拦截`delete proxy[key]`

`ownKeys(target)`拦截`Object.keys(proxy), for...in, Object.getOwnPropertyNames(proxy), Object.getOwnPropertySymbols(proxy)`

`construct(target, args)`用于拦截new命令, 必须返回对象。

`defineProperty`拦截`Object.defineProperty`

`getPropertypeof()`拦截`Object.prototype.__proto__`/`Object.prototype.isPrototypeOf`/`Object.getPrototypeOf`/`Reflect.getPrototypeOf`

`Proxy.revocable()`是可以返回可取消`Proxy`实例`let {proxy, revoke} = Proxy.revocable(target, handler)`，执行`revoke()`则会收回权限。

关于`this`.

一旦`proxy`代理了`target`，那么`target`中的`this`便会指向`proxy`。

### Reflect

可简单理解为`Object`，区别在于之前Object操作失败会报错，而`Reflect`则会返回false。它与`Proxy`的新增方法支持类似。

### object.defineProperty

与`Proxy`的区别是，

1. `Object.defineProperty`只能针对对象单一属性操作，而`proxy`是针对对象自身。

2. `Proxy`可以直接劫持数组


### 数据绑定的实现

```javaScript
var defineReactive = (data) => {
    return new Proxy(data, {
        get(obj, key) {
            // 获取值
            return Reflect.get(obj, key)
        },
        set(obj, key, newval) {
            // 改变值
            Reflect.set(obj, prop, newval)
        }
    })
}
var obj = defineReactive({
    count: 1
})
```
