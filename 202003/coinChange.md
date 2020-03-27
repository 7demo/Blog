# 零钱兑换

> 首先要注意：硬币可能是不规则

> 典型的动态规划。dp[i]代表面值为i时的最小数量，其中dp[0] = 0。假设硬币为：[2,3,5]，那么dp[i] = Math.min(dp[i-2] +1, dp[i-3]+ 1, dp[i-5] + 1)

```javaScript
var coinChange = function(coins, amount) {
    let dp = new Array(amount + 1).fill(Infinity)
    dp[0] = 0
    for (let i = 1; i <= amount; i++ ) {
        for (let k = 0; k < coins.length; k++) {
            if (i - coins[k] >= 0) {
                dp[i] = Math.min(dp[i], dp[i-coins[k]] + 1)
            }
        }
    }
    return dp[amount] == Infinity ? -1 : dp[amount]
};
```
