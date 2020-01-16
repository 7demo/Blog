# 计算排序

> 需要额外的数组空间，要排序的树是一个缺点的范围。时间复杂度为O(n+k)

```javaScript
function CountingSort(arr) {
    let mp = {}
    let ret = []
    for (let i = 0; i<arr.length; i++) {
        let key = arr[i]
        if (mp[key] == undefined) {
            mp[key] = 1
        } else {
            mp[key] ++
        }
    }
    Object.keys(mp).map(key => {
        let val = key
        let count = mp[key]
        while(count) {
            ret.push(val)
            count--
        }
    })
    return ret
}
```
