# reduce

```javaScript
Array.prototype.rd = function() {
    let args = [].slice.call(arguments)
    let fn = args[0]
    let arr = this
    let start = args[1] ? 0 : 1
    let init = args[1] ? args[1] : arr[0]
    for (; start < arr.length; start++) {
        init = fn(init, arr[start], start)
    }
    return init
}
```
