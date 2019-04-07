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
			console.log(i, target-nums[i])
			if (typeof j !== 'undefined') {
				return [j, i]
			}
			obj[nums[i]] = i
		}
	}
```