# 希尔排序

> 是一种优化的插入排序，选择一个增量，然后把数组分为常量个组，每组进行插入排序，不断调整增量。时间复杂度为O(nlog2n)

```javaScript
function ShellSort(arr) {
    let len = arr.length
    let  gap = Math.floor(len / 2)
    while (gap) {
        // 每次
        for (let i = gap; i < len; i++) {
            let tmp = arr[i]
            let index = i
            while (index >=0 && arr[index - 1] > tmp) {
                arr[index] = arr[index - 1]
                arr[index-1] = tmp
                index = index - gap
            }
        }
        gap = Math.floor(gap / 2)
    }
    return arr
}
```

