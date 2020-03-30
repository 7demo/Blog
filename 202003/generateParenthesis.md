# 括号生成

> 回溯法。括号可以新增要注意区分：左括号与右括号，还要判断终止条件。

```javaScript
var generateParenthesis = function(n) {
    let ret = []
    // 当前括号串，左括号长度、右括号长度
    function core(str, l, r, len) {
        if (r > l) {
            return
        }
        if (l == n && r == n) {
            ret.push(str)
            return
        }
        if (!str) {
            core('(', 1, 0, len)
            return
        }
        if (l < n) {
            core(str+'(', l+1, r, len)
        }
        if (r < n) {
            core(str+')', l, r+1, len)
        }
    }
    core('', 0, 0, n)
    return ret
};
```
