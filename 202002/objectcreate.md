# Object.create

### 用法

> 可以接受两个参数，第一个参数为继承的原型链对象，第二个对象会把对象的值直接复制到创建对象内部，而不是原型链上

```javaScript
var pro = {
    p1: '1'
}
var p = Object.create(pro)
var p2 = Object.create(pro, {ps: {value:3}}) // 只能是value，表示值
```

### 实现

```javaScript
Object.create = function(obj) {
    var F = function(){}
    F.prototype = obj
    return new F()
}
```
