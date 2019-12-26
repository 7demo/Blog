# new实现


```javaScript
function new1() {
    let  obj = {}
    let par = Array.prototype.shift.call(arguments)
    // 对象指向原型链
    obj.__proto__ = par.prototype
    // 改变构造函数的this为obj
    let ret = par.apply(obj, arguments)
    //若ret返回值则要特殊处理，特别为null
    return typeof ret === 'object' ? (ret || obj) : obj
}
```

如果用`Object.create`

```javaScript
function new1() {
    let par = Array.prototype.shift.call(arguments)
    let obj = Object.create(par.prototype)
    let ret = par.apply(obj, arguments)
    return typeof ret === 'object' ? (ret || obj) : obj
}
```

其中，`Object.create`与`new Object`的区别在于后者返回的对象有一个`__proto__`属性。
