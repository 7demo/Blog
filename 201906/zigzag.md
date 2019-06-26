# zigzag

Z字形变换，注意遍历时是上下顶点即可。

```javascript
var convert = function(s, numRows) {
    if (!s || s.length <= numRows || numRows < 2) {
        return s
    }
    let flag = true // true表示方向向下 false表示向上
    let map = {}
    let j = 0;
    let res = ''
    for (let i = 0; i < s.length; i++) {
        map[j] = (map[j] || '') + s[i]
        if (flag) {
            j++
            if (j >= numRows) {
                j = numRows - 2
                flag = false
            }
        } else {
            j--
            if (j < 0) {
                j = 1
                flag = true
            }
        }
    }
    for (let i = 0; i < numRows; i++) {
        res += map[i]
    }
    return res
};
```