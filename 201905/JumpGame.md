# Jump Game

```javascript
// 数组中每个值为可以移动的最大步长，是否可以达到最后一个值。时间复杂度为O
// 变量step 为当前的可移动的最大步长。初始化为0.
// 如果最大步长大于等于数组长度，或者最大步长小于索引值（表示永远不能到达这个点，也就是只有达到索引值的点，才能比较reach与跳力能达到的点），跳出循环。
// 如果step 大于等于剩下的未走的步长，则直接拿到结果
function jumpGame(nums) {
    let step = 0;
    const len = nums.length - 1
    for (let i = 0; i <= len; i++) {
        if (step >= len || step < i) {
            break;
        }
        if (step < i + nums[i] ) {
            step = i + nums[i]
        }
    }
    return step >= len
}
```