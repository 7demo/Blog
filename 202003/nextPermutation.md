# 下一个排列

> 比如[5,4,7,5,3,2], 因为是下一个排列，在4与5交换位置后，后面必须是一个倒序数列，直接交换前后位置即可。所以可以变成，找从后到前，顺序的位置，与大于该值得位置，交换后，进行交换位置。

```javaScript
function swap(nums, i, j) {
    let tmp = nums[i]
    nums[i] = nums[j]
    nums[j] = tmp
}
var nextPermutation = function(nums) {
    if (!nums || !nums.length) {
        return
    }
    if (nums.length < 1) {
        return nums
    }
    // 查找倒序的位置
    let next = nums.length - 1
    let pre = next - 1
    while (nums[next] <= nums[pre] && next > 0) {
        pre--
        next--
    }
    let start
    let end = nums.length - 1
    if (next == 0) {
        start = 0
    } else {
        do{
            next++
        } while (next < nums.length && nums[next] > nums[pre])
        swap(nums, pre, next - 1)
        start = pre + 1
    }
    while (start < end) {
        let tmp = nums[end]
        nums[end] = nums[start]
        nums[start] = tmp
        end--
        start++
    }
    return nums
};
```
