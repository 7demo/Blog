# 最长上升子序列

> 动态规划。遍历到i时，在0~i-1找到最大的长度。那么i的长度就是这个长度加1。但是复杂度为O(n^2)

```javaScript
var lengthOfLIS = function(nums) {
    if (!nums || !nums.length) {
        return 0
    }
    let dp = [1]
    let ret = 1
    let arr = [nums[0]]
    for (let i = 1; i < nums.length; i++) {
        let max = 0
        for (let j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                max = Math.max(max, dp[j])
            }
        }
        dp[i] = max + 1
        ret = Math.max(dp[i], ret)
    }
    return ret
};
```

> 贪心算法。
