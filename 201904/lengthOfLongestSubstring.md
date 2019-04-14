# lengthOfLongestSubstring

> 其实就是从字符串中截取不重复的长度最大的字符串

```javascript
var lengthOfLongestSubstring = function(s) {
    if (!s) {
        return 0
    }
    var arr = s.split('')
    var cArr = [] // 把不重复的数字压到数组中
    var res = {} // 存储所有不重复的字符串
    var i = 0
    var len = arr.length
    for (i; i < len; i++) {
        let index = cArr.indexOf(arr[i])
        // 没有重复时
        if (index === -1) {
            cArr.push(arr[i])
        // 遇到重复字符串
        } else {
            res[cArr.length] = cArr.join('')
            cArr.splice(0, index + 1)
            cArr.push(arr[i])
        }
    }
    if (cArr.length) {
        res[cArr.length] = cArr.join('')
    }
    var max = Object.keys(res).reduce((pre, next) => {
        if (typeof pre !== 'number') {
            pre = Number(pre)
        }
        if (typeof next !== 'number') {
            next = Number(next)
        }
        return Math.max(pre, next)
    }, 0)
    return res[max] ? res[max].length : 0
};
```

跑了一下，发现效率有点低。仔细看下了，由于长度是最后遍历数组算的，所以这块我们可以改下长度的获取方式。

```javascript
var lengthOfLongestSubstring = function(s) {
	if (!s) {
		return 0
	}
    var arr = s.split('')
    var cArr = [] // 把不重复的数字压到数组中
    var res = {} // 存储所有不重复的字符串
    var i = 0
    var len = arr.length
    var resLen = 0
    for (i; i < len; i++) {
    	let index = cArr.indexOf(arr[i])
        // 没有重复时
        if (index === -1) {
            cArr.push(arr[i])
        // 遇到重复字符串
        } else {
        	resLen = Math.max(resLen, cArr.length)
        	cArr.splice(0, index + 1)
            cArr.push(arr[i])
        }
    }
    if (cArr.length) {
    	resLen = Math.max(resLen, cArr.length)
    }
    return resLen
};
```