# 防抖与节流

### 如果有函数在等待执行，则会忽略当前

```javaScript
// 非立即执行
function shake(fn, wait) {
    let that = this
    let time
    return function() {
        let args = arguments
        if (time) {
            clearTimeout(time)
        }
        time = setTimeout({
            fn.apply(that, args)
        }, wait)
    }
}
// 立即执行
function shake(fn, wait) {
    let that = this
    let time
    return function() {
        let args = arguments
        if (time) {
            clearTimeout(time)
        }
        let flag = !time

        time = setTimeout({
            time = null
        }, wait)
        if (flag) {
            fn.apply(that, args)
        }
    }
}
```

### 每n秒内执行一次

```javaScript
function throttle(fn, wait) {
    let that = this
    let time = 0
    return function() {
        let args = arguments
        let now = Date.now()
        if (now - time > wait) {
            fn.apply(that, args)
            time = now
        }
    }
}
```
