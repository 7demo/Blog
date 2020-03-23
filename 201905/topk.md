# Top K

> `Top K`是从无序数组中找到最大的K个元素。最直接的算法就是遍历时，通过元素池来进行。时间复杂度为与快排类似。时间复杂度为O(n)，最坏情况为O(n*n)

```javascript
function TopK(arr, k) {
	let tmp = []
	for (let i = 0; i < arr.length; i++) {
		if (tmp.length < k) {
			tmp.push(arr[i])
		} else {
			for (m = 0; m < tmp.length; i++) {
				if (tmp[m] < k) {
					tmp.splice(m, 1, arr[i])
				}
			}
		}
	}
	return tmp
}
```
以上，因为`splice`的关系，所以复杂度是O(n*k*k)。优化点就是`tmp`数据池在比较新元素时，设置边界。但是最好的方法还是采用`quick select`。我们取最大的K个元素，其实是找到最大的K元素，然后拿到右边界k-1个数即是。

```javascript
function TopK(arr, k) {
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
			if (arr[i] > part) {
				swap(arr, index, i)
				index ++
			}
			i++
		}
		return index
	}
	while (start <= end) {
		let partitionIdx = partition(arr, start, end)
		if (k === partitionIdx) {
			return arr.slice(0, partitionIdx)
		}else if(k < partitionIdx) {
            end = partitionIdx - 1;
        } else {
            start = partitionIdx + 1;
        }
	}
	return undefined
}
```

其中切割函数：

```javaScript
function quickSort(nums,left,right){
	var pivot = nums[right];
	while(left<right){
		while(left<right && nums[left]>=pivot){
			left++;
		}
		nums[right]=nums[left];
		while(left<right && nums[right]<pivot){
			right--;
		}
		nums[left]=nums[right];
	}
	nums[right]=pivot;
	return left;
}
```
