# 深拷贝

除却简单拷贝外，需要注意的是对函数、Date、正则、Map、Set、ArrayBuffer、循环引用。

我们先创造一个对象：

```javaScript
var obj = {
    a: 1,
    // b: function(s) {
    // 	console.log(s)
    // },
    c: new Date(),
    d: {
        s: 2
    },
    e: new Set([1]),
    f: /\w/ig,
    m: obj,
    n:  new ArrayBuffer(32)
}
```

### MessageChannel

通过`meeeageChannel`建立两个端口进行数据传输。但是这个操作是异步的，并且不支持函数，并且循环引用时，`key`会为`undefined`

```javaScript
function clone() {
    return new Promise((reslove) => {
        let {port1, port2} = new MessageChannel()
        port2.onmessage = en=>reslove(en.data)
        port1.postMessage(obj)
    })
}
clone(obj).then(data => {
    console.log(data)
})
// a: 1
// c: Mon Jan 06 2020 14:07:56 GMT+0800 (中国标准时间) {}
// d: {s: 2}
// e: Set(1) {1}
// f: /\w/gi
// m: undefined
// n: ArrayBuffer(32) {}
```

### History Api

利用`history.replaceState`。缺点是，路由器可能有限制，也不支持函数。

```javaScript
function clone(obj) {
    let state = history.state
    history.replaceState(obj, document.title)
    let newObj = history.state
    history.replaceState(state, document.title)
    return newObj
}
console.log(clone(obj))
//a: 1
//c: Mon Jan 06 2020 14:16:09 GMT+0800 (中国标准时间) {}
//d: {s: 2}
//e: Set(1) {1}
//f: /\w/gi
//m: undefined
//n: ArrayBuffer(32) {}
```

### Notification

`Notification`也有以上的问题，不过还额外需要用户授权。

### 循环

针对基本类型外的值都进行了特别处理

```javaScript
function clone(val) {
    let tmp = {}
    function isBase(val) {
        return typeof val === 'string' || typeof val === 'number' || typeof val === 'boolean'
    }
    function isSymbol(val) {
        return typeof val === 'symbol'
    }
    function isDate(val) {
        return val instanceof Date
    }
    function isObject(val) {
        return Object.prototype.toString.call(val) === '[object Object]'
    }
    function isSet(val) {
        return Object.prototype.toString.call(val) === '[object Set]'
    }
    function isMap(val) {
        return Object.prototype.toString.call(val) === '[object Map]'
    }
    function isFunction(val) {
        return typeof val == 'function'
    }
    function isReg(val) {
        return val instanceof RegExp
    }
    function baseClone(val) {
        let ret
        if (isBase(val)) {
            ret = vak
        } else if (Array.isArray(val)) {
            ret = [...val]
        } else if (isSymbol(val)) {
            ret = Symbol(val.description)
        } else if (isObject(val)) {
            ret = {...val}
        } else if (isDate(val)) {
            ret = new Date(val)
        } else if (isSet(val)) {
            ret = new Set(...val)
        } else if (isMap(val)) {
            ret = new Map()
            val.forEach((key, value) => {
                ret.set(key, value)
            })
        } else if (isFunction(val)) {
            ret = new Function('return ' + val.toString())()
        } else if (isReg(val)) {
            let pattern = val.valueOf()
            let flags = ''
            flags += pattern.global ? 'g' : ''
            flags += pattern.ignoreCase ? 'i' : ''
            flags += pattern.multiline ? 'm' : ''
            ret = new RegExp(pattern.source, flags)
        }
        Reflect.ownKeys(ret).forEach(key => {
            if (typeof ret[key] === 'object' && ret[key] != null) {
                if (tmp[ret[key]]) {
                    ret[key] = tmp[ret[key]]
                } else {
                    tmp[ret[key]] = ret[key]
                    ret[key] = baseClone(ret[key])
                }
            }
        })
        return ret
    }
    return baseClone(val)
}
```
