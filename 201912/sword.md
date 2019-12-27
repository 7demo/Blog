# 剑指

## 基本概念

#### 数组

数组与字符串是连续内存，要预先分配，空间性不好，因次有动态数组，每次分配较小内存，不足时，重新分配一块大的空间，原内存空间释放，但是比较耗时。

因为内存连续的原因，数组的读取是O(1),


## 算法

#### 3. 找到数组重复的数字

> 3.0 在长度为n的数组中，所有值都在0 ~ n-1间

*方案一* 先排序，再比较相邻是否一样，时间复杂度为O(nlogn)

*方案二* 利用哈希表，复杂度为O(n), 但是空间复杂度为O(n)

*方案三* 如果数组不存在重复、且有序的情况下，数组中的值必定和该值得索引相同，如果不同则拿该值与以该值为索引的值进行比较，如果相同则为重复值，如果不同，则移动该值到对应的索引位置。空间复杂度为O(1), 时间复杂度为O(n)

```javaScript
function getRepeat(arr, start = 0) {
    let ret = []
    for (let i = start; i < arr.length; i++) {
        // 当前索引值
        let value = arr[i]
        // 以当前索引值为索引的 值
        let valueByvalue = arr[value]
        // 相同，则是有序不重复数组，则无问题
        // 不同 则若两者一样，则需要移动位置，重新计算比较当前索引的值
        if (value !== i) {
            if (value === valueByvalue) {
                ret.push(value)
            } else {
                arr[i] = valueByvalue
                arr[value] = value
                i--
            }
        }
    }
    return ret
}
```

当不能修改源数组时，改如何操作？

> 3.1 长度为n+1的数组，值都在1~n范围内。

*方案一* 复制数组如上操作，但是空间复杂度为O(n).

*方案二* 如果避免使用辅助空间，如果长度为n+1, 取其中间数m，分为大于m的一组，小于m的一组，如果切分的数组的长度超过了m至另外一端的值，那就是有重复的。空间复杂度为O(1), 时间复杂度为O(nlogn)

```javaScript
// 遍历整个数组，获得start-end之间元素的个数
function getCount(arr, len, start, end) {
    let count = 0
    for (i = 0; i < len; i++) {
        if (start <= arr[i] && arr[i] <= end) {
            count ++
        }
    }
    return count
}
function getRepeat(arr) {
    let end = arr.length - 1
    let len = arr.length
    let start = 1
    while (end >= start ) {
        let mid = Math.floor((end + start) / 2)
        // 拿到区间元素个数进行比较，如果大于中间值，则必有重复
        // 否则另外一个区间有重复
        // 区间为二分查找，而每次又需要遍历寻找个数，所以为logn * n
        let count = getCount(arr, len, start, mid)
        if (end == start) {
            if (count) {
                return count
            }
        }
        if (count > mid - start + 1) {
            end = mid
        } else {
            start = mid + 1
        }
    }
    return ret
}
```

#### 4. 二维数组是否包含给予的整数

> 二维数组每行都是从左到右递增，从上到下也是递增

```
1 2 8 9
2 4 9 12
4 7 10 13
6 8 11 15
```
二维数组可以视为一个方格，第一次取右上角的值，如果比目标值大，说明最有一列直接可以舍弃。如果比目标值小，说明要从第二行查找。

```javaScript
function find(arr, num) {
    let i = 0
    let j = arr[0].length - 1
    while (i < arr.length && j >= 0) {
        if (arr[i][j] === num) {
            return true
        }
        if (arr[i][j] > num) {
            j --
        } else {
            i++
        }
    }
    return false
}
```
