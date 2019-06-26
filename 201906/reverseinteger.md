# Reverse Integer

只需要注意越界即可.

```javascript
let isFlag = false
if (x < 0) {
	isFlag = true
}
x = Math.abs(x) + ''
let res = 0
for (let i = x.length - 1; i > -1; i--) {
	res += x[i] * Math.pow(10, i)
}
if (res > Math.pow(2, 31) - 1) {
	return 0
}
if (isFlag) {
	res = -(res)
}
return res
```