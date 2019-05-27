# Jump Game 2

```javascript
// 找出跳出数组最短的操作
// 从索引为0到当前步力可以达到的地方进行循环。
// 当前索引的值与该步可以达到最远的地方，与 当前总步力取最大值。
function jump(nums) {
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