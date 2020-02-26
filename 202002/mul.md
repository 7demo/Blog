# 连乘函数

> 要求实现：mul(a)(m)(c)

> 思路：1，肯定返回函数，2，输出其实是改造tostring

```javaScript
function mul(x) {
    let fn = function(y) {
        return mul(x*y)
    }
    fn.toString = function() {
        return x
    }
    return fn
}
```
