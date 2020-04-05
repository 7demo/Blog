# 打家劫舍

> 动态规划。

```javaScript
var rob = function(nums) {
    if (!nums || !nums.length) {
        return 0
    }
    let arr = []
    arr[0] = nums[0]
    arr[1] = Math.max(nums[1], nums[0])
    arr[2] = Math.max(nums[1], nums[0]+nums[2])
    for (let i = 3; i < nums.length; i++) {
        arr[i] = Math.max(nums[i] + arr[i-2], nums[i] + arr[i-3], arr[i-1])
    }
    return arr[nums.length - 1]
};
```
