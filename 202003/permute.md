# 全排列

```javaScript
var permute = function(nums) {
    let ret = []
    let len = nums.length
    function r(arr = [], tmp = []) {
        if (tmp.length == len) {
            ret.push(tmp)
        } else {
            for (let i = 0; i < arr.length; i++) {
                let _arr = [...arr]
                let _tmp = [...tmp]
                _tmp.push(_arr[i])
                _arr.splice(i, 1)
                r(_arr, _tmp)
            }
        }
    }
    r(nums)
    return ret
}
```
