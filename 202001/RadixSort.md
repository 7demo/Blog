# 基础排序

> 找到数组中最大值，找到其位数（转化为字符串的长度），然后从低位到高位进行桶排序，每此桶都为0-9。比如个位排序时，是把所有数字基于个位数进行排序，然后依据上次装进桶的结果进行十位、百位进行排序。时间复杂度为O(N*K)，其中K为位数。

```javaScript
// 根据
function getNumber(num, radix) {
    radix = radix + 1
    let str = num + ''
    if (str.length < radix) {
        return 0
    }
    let retNum = str.substr(str.length - radix, 1) || 0
    return Number(retNum)
}
function RadixSort(arr) {
    let max = 0
    let ret = []
    for (let i = 0; i < arr.length; i++) {
        max = Math.max(max, arr[i])
    }
    let radix = (max + '').length
    let bucket = []
    for (let i = 0; i <= radix; i++) {
        bucket[i] = bucket[i] || []
        let sourceArr = i > 0 ? bucket[i - 1] : arr
        for (let j = 0; j < sourceArr.length; j++) {
            let ele = sourceArr[j]
            if (ele == undefined) {
                continue
            }
            if (ele instanceof Array) {
                for (let k = 0; k < ele.length; k++) {
                    let num = getNumber(ele[k], i)
                    if (i + 1 <= radix) {
                        bucket[i][num] = bucket[i][num] || []
                        bucket[i][num].push(ele[k])
                    } else {
                        ret.push(ele[k])
                    }
                }
            } else {
                let num = getNumber(ele, i)
                if (i + 1 <= radix) {
                    bucket[i][num] = bucket[i][num] || []
                    bucket[i][num].push(ele)
                } else {
                    ret.push(ele)
                }
            }
        }
    }
    return ret
}
```
