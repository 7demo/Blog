# 有效的括号

> 是根据一个栈，先进后出。

```javaScript
function isLeft(s) {
    if (s == '(' || s == '[' || s == '{') {
        return true
    }
}
function isOk(s1, s2) {
    if (s1 == '(' && s2 == ')') {
        return true
    }
    if (s1 == '[' && s2 == ']') {
        return true
    }
    if (s1 == '{' && s2 == '}') {
        return true
    }
}
var isValid = function(s) {
    let i = 0
    let len = s.length
    let l = 0
    let tmp = []
    for (; i < len; i++) {
        if (isLeft(s[i])) {
            tmp.unshift(s[i])
        } else {
            if (!isOk(tmp.shift(), s[i])) {
                return false
            }
        }
    }
    if (tmp.length) {
        return false
    }
    return true
};
```
