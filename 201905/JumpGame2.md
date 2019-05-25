# Jump Game 2

```javascript
// 数组中每个值为可以移动的最大步长，最少移动次数
// 变量step 为当前的可移动的最大步长。初始化为0. len表示数组长度. count表示移动几次，i 表示遍历的数组的位置。
// 如果最大步长还小于数组长度，代表还没跳出数组。则开始跳一次。在这次跳的时候，则从当前遍历的索引i，到当前步长的可达到位置。然后取现在的步长与索引位置可以达到的最大位置。
// 时间复杂度为 O(n)
function jumpGame(nums) {
    let step = 0;
    const len = nums.length - 1;
    let count = 0
    let i = 0;
    while (step < len) {
    	count ++
    	let _pre = step
    	for (; i <= _pre; i++) {
    		step = Math.max(step, i + nums[i])
    	}
    }
    return count
}
```