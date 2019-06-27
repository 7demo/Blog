# Palindrome Number

```javascript
var isPalindrome = function(x) {
    if (x < 0) {
        return false
    }
    x = x + ''
    let left = 0
    let right = 0
    // 偶数
    if (x.length % 2 == 0) {
        left = x.length / 2 - 1
        right  = x.length / 2
    } else {
        left = (x.length - 1) / 2 - 1
        right = (x.length + 1) / 2
    }
    for (; left > -1 ; left--, right++) {
        if (x[left] != x[right]) {
            return false
        }
    }
    return true
};
```