# String to Integer

```javascript
var myAtoi = function(str) {
	if (!str) {
		return 0
	}
	let flag = false // 是否负数
	let res = 0
	str = str.trim()
	if (str[0] == '+') {
		str = str.substr(1)
	} else if (str[0] == '-') {
		flag = true
		str = str.substr(1)
	}
	let reg = /^(\d+)/
	if (reg.test(str)) {
		res = RegExp.$1 || 0
		if (res >= Math.pow(2, 31)) {
			if (flag) {
				res = Math.pow(2, 31)
			} else {
				res = Math.pow(2, 31) - 1
			}
		}
		if (flag) {
			res = -(res)
		}
	}
	return res
}
```