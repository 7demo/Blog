# longestPalindrome

> 参考：马拉车算法

```javascript
var longestPalindrome = function(a) {
    let t = '$#'
    for (let i = 0; i < a.length; i++) {
        t += a[i]
        t += '#'
    }
    let p = Array.from({length: t.length})
    let id = 0
    let mx = 0
    let resId = 0
    let resMx = 0
    for (let i = 1; i < t.length; i++) {
        p[i] = mx > i ? Math.min(p[2 * id - 1], mx - 1) : 1
        while (t[i + p[i]] == t[i - p[i]]) {
            ++p[i]
        }
        if (mx < i + p[i]) {
            mx = i + p[i]
            id = 1
        }
        if (resMx < p[i]) {
            resMx = p[i]
            resId = i
        }
    }
    return a.substr((resId - resMx) / 2, resMx - 1)
};
```