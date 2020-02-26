# flat实现

> 特点：如果是空元素将会清除，默认深度为1

```javaScript
// reduce实现
Array.prototype.ft1 = function() {
    let arr = this
    let args = [].slice.call(arguments)
    let deep = args[0] ? args[0] : 1
    if (deep > 1) {
        return arr.reduce((pre, cur, i) => {
            if (Array.isArray(cur)) {
                pre = pre.concat(cur.ft1(deep - 1))
            } else {
                pre = pre.concat(cur)
            }
            return pre
        }, [])
    } else if (deep == 1) {
        return arr.reduce((pre, cur, i) => {
            return pre.concat(cur)
        }, [])
    }
}
```
