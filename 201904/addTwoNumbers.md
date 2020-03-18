# add Two Numbers

> 虽然难度是中级，但是题目却有点绕。到底我们的输入与输出是什么样的一个对象？

## 链表

链表：`linked-list`。可以分为：单向链表、双向链表、单项循环链表、双向循环链表。

在单向链表中，每个节点都是一个包含有值与`next`属性的对象。单个节点如下：

```javascript
function NodeList(val) {
	this.val = val
	this.next = null
}
```

`next`属性指向下一个节点。如此反复，组成一个单向链表。如下：

```javascript
var l1 = {
    val: 2,
    next: {
        val: 4,
        next: {
            val: 3,
            next: null
        }
    }
}
var l1 = {
    val: 5,
    next: {
        val: 6,
        next: {
            val: 4,
            next: null
        }
    }
}
```

至于单向循环链表，则会在对后一个节点的`next`指向第一个节点。

双向链表，则是每个节点不但有`next`属性，还有一个`previous`指向它的父节点。


## add Two Numbers

这个算法要求输入的就是两个单向链表，如上面的例子`l1`与`l2`，输出是`l3`：

```javascript
var l1 = {
    val: 7,
    next: {
        val: 0,
        next: {
            val: 8,
            next: null
        }
    }
}
```

那具体解法是：

```javascipt
var addTwoNumbers = function(l1, l2) {
    let l3 = new ListNode() // 返回结果的父节点，也是第一个节点
    let tmp = new ListNode()
    let ex = 0
    l3.next = tmp // 表示子节点，也是当前处理节点
    while (l1 || l2 || ex) {
        let v1 = l1 ? l1.val : 0
        let v2 = l2 ? l2.val : 0
        let v3 = v1 + v2 + ex
        if (v3 > 9) {
            v3 = v3 % 10
            ex = 1
        } else {
            ex = 0
        }
        tmp.val = v3
        l1 = l1 ? l1.next : null
        l2 = l2 ? l2.next : null
        // 如果继续循环，则此对象新增子对象，并把当前对象变量指向子对象
        if (l1 || l2 || ex) {
            tmp.next = new ListNode()
            tmp = tmp.next || null
        }
    }
    return l3.next
};
```

```javaScript
function ListNode(val) {
		    this.val = val;
		    this.next = null;
		}

		var addTwoNumbers = function(l1, l2) {
			let m1 = ''
			let m2 = ''
			while (l1) {
				m1 = (l1.val + '') + m1
				l1 = l1.next
			}
			while (l2) {
				m2 = (l2.val + '') + m2
				l2 = l2.next
			}
			let len1 = m1.length - 1
			let len2 = m2.length - 1
			let res = ''
			let mus = 0
			while (len1 > -1 || len2 > -1) {
				let s1 = m1[len1]/1 || 0
				let s2 = m2[len2]/1 || 0
				let ms = s1 + s2 + mus
				if (ms > 9) {
					mus = 1
					ms = ms%10
				} else {
					mus = 0
				}
				res = (ms + '') + res
				len1--
				len2--
			}
			if (mus) {
				res = (mus + '') + res
			}
			let len = res.length - 1
			let ret
			let cur
			while (len > -1) {
				let nnode = new ListNode(res[len])
				if (ret) {
					cur.next = nnode
					cur = nnode
				} else {
					ret = nnode
					cur = nnode
				}
				len--
			}
			return ret
		}
```
