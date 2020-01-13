# 堆排序

> 堆类似一个完全二叉树，子节点总大于或者小于父节点。时间复杂度为O(nlogn)

> 大根堆：每个节点都大于左右子节点的值，称为大根堆。

> 小根堆：每个节点都小于左右节点的值，称为小根堆。

另外，查找数组中的父节点或者左右子节点，如果某个值已知索引为i，那么父节点的索引为(i-1)/2，左子节点的索引为2*i + 1, 右子节点的索引为2*i + 2.所以：

```
arr[i] > arr[2*i + 1]; arr[i] > arr[2*i +2] // 大根堆

arr[i] < arr[2*i + 1]; arr[i] < arr[2*i +2] // 小根堆
```

采用大根堆的形式：

```javaScript
function HeapSort(arr) {
    let len = arr.length
    makeMaxHeap(arr, len)
    while (len) {
        swap(arr, 0, len - 1)
        len --
        makeMaxHeap(arr, len)
    }
    return arr
}

function makeMaxHeap(arr, len) {
    for (let i =1; i < len; i++) {
        let current = i
        let parent = Math.floor((current - 1) / 2)
        while (arr[current] > arr[parent]) {
            console.log(current, parent, arr)
            swap(arr, current, parent)
            current = parent
            parent =  Math.floor((current - 1) / 2)
        }
    }
}

function swap(arr, i, j) {
    let tmp = arr[j]
    arr[j] = arr[i]
    arr[i] = tmp
}

```
