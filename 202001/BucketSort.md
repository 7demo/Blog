# 桶排序

> 是计数排序的升级版本。把输入数据分到有限数量的桶里，每个桶再进行排序。计数排序是统计每个数的出现次数，而桶则是一个区间。

```
function BicketSort(arr, bucketSize) {
    // 如果数组长度少于2 直接返回
    if (arr.length < 2) {
        return arr
    }
    let len = arr.length
    let min = 0
    let max = 0
    let ret = []
    // 默认情况下每个桶的数量最大可以装10个，所以在递归的时候每个桶可以装的数量必须递减，否则可能爆栈。
    bucketSize = bucketSize || 10
    // 获得最大值 最小值
    for (let i = 0; i < len; i++) {
        min = Math.min(arr[i], min)
        max = Math.max(arr[i], max)
    }
    let bucketArr = []
    // 桶的个数，如果桶中有重复的数字的话，bucketCount可能为1
    let bucketCount = Math.floor(arr.length, Math.floor((max - min) / bucketSize) + 1)
    // 遍历数组，把值装进对应的桶
    for (let i = 0; i < len; i++) {
        let index = Math.floor((arr[i] - min) / bucketSize)
        bucketArr[index] = bucketArr[index] || []
        bucketArr[index].push(arr[i])
    }
    for (let i = 0; i < bucketArr.length; i++) {
        bucketArr[i] = bucketArr[i] || []
        // 如果桶有重复数字的情况
        if (bucketCount == 1) {
            for (let j = 0; j < bucketArr[i].length; j++) {
                ret.push(bucketArr[i][j])
            }
        } else {
            // 可能发生，桶中可以装的数字与数组的长度，那么就取最小值。避免桶可以装的个数大于数组的长度，都分配到一个桶，造成递归
            ret.push(...BicketSort(bucketArr[i], Math.min(bucketSize, bucketArr[i].length)))
        }
    }
    return ret
}
```
