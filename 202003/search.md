# 搜索旋转排序数组

```javaScript
if (!nums || !nums.length) {
        return -1
    }
    if (nums.length == 1) {
        if (nums[0] == target) {
            return 0
        } else {
            return -1
        }
    }
    if (target < nums[0] && target > nums[nums.length - 1]) {
        return -1
    }
    let start = 0
    let end = nums.length - 1
    while (start < end) {
        let mid = (start + end) >> 1
        let midVal = nums[mid]
        let startVal = nums[start]
        let midr = mid + 1
        let midrVal = nums[midr]
        let endVal = nums[end]
        if (midVal == target || target == midrVal) {
            if (midVal == target) {
                return mid
            }
            if (midrVal == target) {
                return midr
            }
        } else {
            // 乱序
            if (startVal > endVal) {
                if(endVal > target) {
                    start = mid
                } else {
                    end = mid
                }
            // 正序
            } else {
                if(midVal > target) {
                    start = mid
                } else {
                    end = mid
                }
            }
        }
    }

    return -1
};
```

以上执行超时。

```javaScript
if (!nums || !nums.length) {
        return -1
    }
    if (nums.length == 1) {
        if (nums[0] == target) {
            return 0
        } else {
            return -1
        }
    }
    if (target < nums[0] && target > nums[nums.length - 1]) {
        return -1
    }
    let start = 0
    let end = nums.length - 1
    while (start <= end) {
        let mid = (start + end) >> 1
        let midVal = nums[mid]
        let startVal = nums[start]
        let endVal = nums[end]
        if (midVal == target) {
            return mid
        }
        // 是顺序
        if (startVal <= midVal) {
            if (target >= startVal && target < midVal) {
                end = mid - 1
            } else {
                start = mid + 1
            }
        // 乱序数组
        } else {
            if (target <= endVal && target > midVal) {
                start = mid + 1
            } else {
                end = mid - 1
            }
        }
    }
    return -1
```
