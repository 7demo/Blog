# bind方法


## bind的实现

```javaScript
Function.prototype.bind = function() {
    let fun = this
    // bind 的 arguments
    let args = Array.prototype.slice.apply(arguments)
    let context = args.shift()
    let arg = args.slice()
    let fnop = function(){}
    let fbound = function() {
        // 执行时的arguments
        arg = arg.concat(Array.prototype.slice.apply(arguments))
        return fun.apply(context, arg)
    }
    // 此操作主要是为了最终返回的函数的原型链与原函数相同
    fnop.prototype = this.prototype
    fbound.prototype = new fnop()
    return fbound
}
```

## apply的实现

> apply与call类似，区别在于参数。

```javaScript
Function.prototype.apl = function() {
    let args = Array.prototype.slice.apply(arguments)
    let context = args[0] || window
    context.fn = this
    let arg = args[1] || []
    let result = eval('context.fn('+ arg +')')
    delete result.fn
    return result
}
```
