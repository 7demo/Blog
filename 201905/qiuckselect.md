# qiuck select

> `qiuck select`是从无序数组中找到第K小的元素。与快排类似。时间复杂度为O(n)，最坏情况为O(n*n)

```javascript
function qiuckSelect(arr, k) {
	let start = 0;
	let end = arr.length - 1
	function swap(arr, o, p) {
		let tmp = arr[o]
		arr[o] = arr[p]
		arr[p] = tmp
	}
	function partition(arr, i, j) {
		let part = arr[i] // 基准值
		let index = i // 基准值的索引
		while (i <= j) {
			// 如果值小于基准值，则移动到左边去
			if (arr[i] < part) {
				swap(arr, index, i)
				index ++
			}
			i++
		}
		return index
	}
	while (start <= end) {
		let partitionIdx = partition(arr, start, end)
		if ((k - 1) === partitionIdx) {
			return arr[partitionIdx]
		}else if((k - 1) < partitionIdx) {
            end = partitionIdx - 1;
        } else {
            start = partitionIdx + 1;
        }
	}
	return 0
}
```

思路：

1，类似于快排，选择一个基准值，遍历数组，比此数组小的元素放到左侧。每放一次，则累加一，得到`partitionIdx`。

2，如果所求K值与`partitionIdx`相等，则就是所求值。否则则调整遍历的开始与结束位置，继续循环。