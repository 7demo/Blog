# 图解算法

> `O(log n)`对数时间，有二分查找。

> `O(n)`线性时间，有简单查找。

> `O(n * log n)`有快速查找。

> `O(n*n)`有选择排序。

> `O(!n)`有旅行商的查找。

## 二分查找

二分法的复杂度`O(n)`，它的前提条件是：有序元素列表。查找目标元素的最大次数为`log2 n`。

```javascript
/**
 * 二分法
 * @param  {[type]} array [目标对列]
 * @param  {[type]} ele   [目标元素]
 * @param  {[type]} low   [索引]
 * @return {[type]}       [description]
 */
function binarySearch(array, ele, low) {
	low = low || 0
	let index = Math.floor(array.length / 2)
	if (array[index] === ele) {
		return index + low
	} else if (array[index] > ele) {
		return binarySearch(array.slice(0, index), ele, low)
	} else if (array[index] < ele) {
		return binarySearch(array.slice(index, array.length), ele, index)
	}
	return -1
}
```

也可以不切数组，只改变下标的方式来做。

```javascript
/**
 * 二分法
 * @param  {[type]} array [目标对列]
 * @param  {[type]} ele   [目标元素]
 * @return {[type]}       [description]
 */
function binarySearch(array, ele) {
	let low = 0
	let end = array.length
	while (low < end) {
		let mid = low + Math.floor((end - low) / 2)
		let val = array[mid]
		if (val === ele) {
			return mid
		} else if (val > ele) {
			end -= mid
		} else {
			low += mid
		}
	}
	return -1
}
```