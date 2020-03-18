# 三数之和

> 三树之和转为二数之和，要求和为0，此处测试案例比较复杂，有倒序、重复、全部为0、长度不够，内存溢出的情况。

```
function twoSum(nums, target, index) {
    let obj = {}
    let i
    let j
    let len = nums.length
    let ret = []
    for (i = 0; i < len; i++) {
        if (index == i) {
            continue
        }
        j = obj[target - nums[i]]
        if (typeof j !== 'undefined') {
            ret.push([j, i])
        }
        obj[nums[i]] = i
    }
    return ret
}
function QucikSort(arr) {
    if (arr.length < 2) {
        return arr
    }
    let mid = Math.floor(arr.length / 2)
    let baseVal = arr[mid]
    let left = []
    let right = []
    let eq = []
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] < baseVal) {
            left.push(arr[i])
        } else if(arr[i] > baseVal) {
            right.push(arr[i])
        } else {
            eq.push(arr[i])
        }
    }
    return QucikSort(left).concat(eq, QucikSort(right))
}
function threeSum(nums) {
    if (nums.length < 3) {
        return []
    }
    let ret = []
    let len = nums.length
    let i = 0
    let obj = {}
    for (; i < len; i++) {
        if (nums[i] == nums[i-1]) {
            continue
        }
        let tmp = nums[i]
        let k = 0 - tmp
        let res = twoSum(nums, k, i)
        if (res.length) {
            let m = 0;
            let mlen = res.length
            for (; m < mlen; m++) {
                let sortRes = QucikSort([tmp, nums[res[m][0]], nums[res[m][1]]])
                let key = sortRes.join('-')
                if (!obj[key]) {
                    ret.push(sortRes)
                    obj[key] = 1
                }
            }
        }
    }
    return ret
}
```

> 以下代码时普通代码，如果是0的话，可以根据0对数组进行分类

```javaScript
function threeSum(nums) {
    if (nums.length < 3) {
        return []
    }
    nums = QucikSort(nums)
    if (nums[0] > 0 && nums[nums.length - 1] < 0) {
        return []
    }
    let ret = []
    let len = nums.length
    let i = 0
    let obj = {}
    for (; i < len; i++) {
        if (nums[i] === nums[i-1]) {
            continue
        }
        let l = i + 1
        let r = len - 1
        while (l < r && nums[i] < 1) {
            let sum = nums[i] + nums[l] + nums[r]
            if (sum === 0) {
                ret.push([nums[i], nums[l], nums[r]])
                l++
                r--
                while (l < r && nums[l] == nums[l-1]) {
                    l++
                }
                while (l < r && nums[r] == nums[r+1]) {
                    r--
                }
            } else if (sum > 0) {
                r--
            } else {
                l++
            }
        }
    }
    return ret
}
```
