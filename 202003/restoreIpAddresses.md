# 复原ip

> 利用回溯法，类似一组字符串的全排列

```javaScript
var restoreIpAddresses = function(s) {
    if (!s || s.length < 4 || s.length > 12) {
        return []
    }
    let ret = []
    function r(str, len, tmp = []) {
        if (tmp.length == 4 && str == '') {
            ret.push(tmp.join('.'))
        }
        if (str.length > len * 3) {
            return
        }
        if (str.length < len) {
            return
        }
        // 只有一个值
        let _tmp1 = [...tmp]
        _tmp1.push(str.substr(0, 1))
        r(str.substr(1), len - 1, _tmp1)
        if (str.length > 1 && str[0] != '0') {
            // 只有两个值
            let _tmp2 = [...tmp]
            _tmp2.push(str.substr(0, 2))
            r(str.substr(2), len - 1, _tmp2)
        }
        if (str.length > 2 && str.substr(0, 3) < '256' && str[0] != '0') {
            // 只有两个值
            let _tmp3 = [...tmp]
            _tmp3.push(str.substr(0, 3))
            r(str.substr(3), len - 1, _tmp3)
        }
    }
    r(s, 4)
    return ret
};
```
