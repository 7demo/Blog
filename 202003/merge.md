# 合并数组空间

> 此题必须先对数组进行排序，否则针对[[2,3],[4,5],[6,7],[8,9],[1,10]]无法通过。

```javaScript
function quickSort(arr) {
    if (!arr) {
        return []
    }
    let len = arr.length
    if (len < 2) {
        return arr
    }
    let i = 0
    let midIndex = (len + i) >> 1
    let baseArr = arr[midIndex]
    let base = baseArr[0]
    let left = []
    let mid = [baseArr]
    let right = []
    for (; i < len; i++) {
        if (arr[i][0] < base) {
            left.push(arr[i])
        } else if (arr[i][0] > base) {
            right.push(arr[i])
        } else {
            mid.push(arr[i])
        }
    }
    return quickSort(left).concat(mid, quickSort(right))
}
var merge = function(intervals) {
    if (!intervals || !intervals.length) {
        return []
    }
    if (intervals.length == 1) {
        return intervals
    }
    intervals = quickSort(intervals)
    let ret = []
    let pre = intervals[0]
    for (let i = 1; i < intervals.length; i++) {
        let cur = intervals[i]
        // 保证 pre开始小，cur开始大
        if (pre[0] >= cur[0]) {
            let tmp = cur
            cur = pre
            pre = tmp
        }
        if (pre[1] < cur[0]) {
            ret.push(pre)
            pre = cur
        } else if (pre[1] == cur[0]) {
            pre[1] = cur[1]
        } else {
            if (pre[1] <= cur[1]) {
                pre[1] = cur[1]
            }
        }
    }
    if (pre.length) {
        ret.push(pre)
    }
    return ret
};
```
