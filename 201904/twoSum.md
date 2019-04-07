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

以上是`leetcode`能接受的答案，以空间换时间。如果只单纯实现其实还有：

```javascript
var twoSum = function(nums, target) {
    let run = (nums, target) => {
        var res = nums.reduce((cur, next) => {
            if (cur + next === target) {
                return [cur, next]
            } else {
                return cur
            }
        })
        if (!Array.isArray(res)) {
           return run(nums.slice(1), target)
        } else {
           return res
        }
    }
    const valurArr = run(nums, target)
    return [nums.indexOf(valurArr[0]), nums.indexOf(valurArr[1])]
};
```
以上，虽然已经为了避免堆栈过长，使用了“尾递归”来优化，但是还是没有被通过。