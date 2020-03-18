# Two Sum

> 给定一个数组与一个目标值，数组中存在两个值相加等于目标值，求两个值的在数组中的位置。

```javascript
	var twoSum = function(nums, target) {
		let obj = {}
		let i
		let j
		let len = nums.length
		for (i = 0; i < len; i++) {
			j = obj[target - nums[i]]
			if (typeof j !== 'undefined') {
				return [j, i]
			}
			obj[nums[i]] = i
		}
	}
```

是`leetcode`能接受的答案，额外使用了一个`O(N)`的空间。如果不让使用额外空间也可以：

```javaScript
function twoSum(nums, target) {
    let start = 0
    let end = nums.length - 1
    while (start < end) {
        let res = nums[start] + nums[end]
        if (res == target) {
            return [start, end]
        } else if (res > target) {
            end --
        } else if (res < target) {
            start ++
        }
    }
    return []
}
```

这个方法也是`O(N)`，不过当数组时一个倒序数组或者数组满足的值没有分布在前后两端就会出问题。
