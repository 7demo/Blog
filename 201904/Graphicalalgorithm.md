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

## 选择排序

选择排序，每次从剩余数组中取出最大的值。运行时间为`O(n*n)`。

```javascript
/**
 * 选择排序
 * @param  {[type]} array [目标对列]
 * @return {[type]}       [description]
 */
function selectSort(array) {
	let arr = []
	while (array.length) {
		let max = 0
		let index = 0
		for (let i = 0; i < array.length; i++) {
			if (array[i] > max) {
				max = array[i]
				index = i
			}
		}
		array.splice(index, 1)
		arr.push(max)
	}
	return arr
}
```

## 递归

递归因为调用栈的原因，一般都会出现堆栈溢出的问题。比如阶乘：

```javascript
function factorial(n) {
	if (n == 1) {
		return 1
	} else {
		return n * factorial(n - 1)
	}
}
```

常见的优化方法是`尾调用`。尾调用是指函数执行的最后一步是返回一个函数，这个过程中不是增加一个新栈，而是更新。使用尾调用则为：

```javascript
function factorial(n, m = 1) {
	if (n == 1) {
		return m
	} else {
		return factorial(n - 1, n * m)
	}
}
```

不过由于v8引擎未开尾调用的原因，尾调用优化后的递归无效。我们可以使用循环来试下：

```javascript
function factorial(n) {
	var val = 1
	while (n) {
		val *= n
		n--
	}
	return val
}
```

或者使用`蹦床函数`:

```JavaScript

```
