# 希尔排序

> 是一种优化的插入排序，选择一个增量，然后把数组分为常量个组，每组进行插入排序，不断调整增量。时间复杂度为O(nlog2 n)——此处2为底数

> 第一次是n/2个组，每个组需要比较1次

> 第二次是n/4个组，每个组需要比较 3+2+1次

> 第三次是n/8个组，，每个组需要比较 7+6+...+1次

所以可以得到第n/(2^m)次，每组需要比较2^(m-1)与2^(m)之间。

就研究表明，可以适当增加常量步长，减少扫描次数。比如分组个数为1/3, 1/3^2。最有情况为O(n^(7/6))

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

