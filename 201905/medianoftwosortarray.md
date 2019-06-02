# Median of two sort array

+ 方案一：

> 两个数组合并成一个数组，排序。然后获取这个合并后数组的中位数。使用快排，时间复杂度为：O((m+n) * logn)。

+ 方案二：

> 时间复杂度为 O(log (m + n))，也是`leetcode`要求的。一看要求的时间复杂度，就应该想到二分法。

- 两个数组区分长度，a小,b大。

- a中二分取中间值，比较此值与b的中间值大小。小于的话，去掉a中取消a至开始与b中结束值往前推中间值-开始值。



```javascript
var findMidInOddArray = function(a, b) {
    let begain_a = 0
    let begain_b = 0
    let end_a = a.length - 1
    let end_b = b.length -1
    function get(a, begain_a, end_a, b, begain_b, end_b) {
    	// 终止条件
    	if (end_a - begain_a == 1) {
    		if (a[end_a] >= b[Math.floor((begain_b + end_b) / 2)]) {
    			return  b[Math.floor((begain_b + end_b) / 2)]
    		} else {
    			return Math.max(a[end_a],  b[Math.floor((begain_b + end_b) / 2) - 1])
    		}
    	}

    	if (end_a - begain_a === 0) {
    		if ((end_b - begain_b) % 2 === 1) { // 偶数
    			if (a[begain_a] < b[Math.floor((begain_b + end_b)/2)]) {
    				return b[(begain_b + end_b)/2]
    			} else {
    				return Math.min(a[begain_a], b[Math.floor((begain_b + end_b)/2) + 1])
    			}
    		} else {
    			if (a[begain_a] >= b[Math.floor((begain_a + begain_b)/2)]) {
    				return b[Math.floor((begain_a + begain_b)/2)]
    			} else {
    				return Math.min(a[begain_a], b[Math.floor((begain_a + begain_b)/2) - 1])
    			}
    		}
    	}


    	let mid_a = Math.floor((begain_a + end_a) / 2)
    	let mid_b = Math.floor((begain_b + end_b) / 2)
    	if (a[mid_a] < b[mid_b]) {
    		return get(a, mid_a, end_a, b, begain_a, end_b - (mid_a - begain_a))
    	} else if (a[mid_a] > b[mid_b]) {
    		return get(a, begain_a, mid_a, b, begain_b + (end_a - mid_a), end_b)
    	} else {
    		return a[mid_a]
    	}
    }
    if (a.length < b.length) {
    	return get(a, begain_a, end_a, b, begain_b, end_b)
    } else {
    	return get(b, begain_b, end_b, a, begain_a, end_a)
    }
};
```


参考：

[中位数](https://v.youku.com/v_show/id_XMzg0NDQwMjg5Ng==.html?spm=a2h0k.11417342.soresults.dtitle)