# 链表相加

> 借助栈

```javaScript
function ListNode(val) {
    this.val = val;
    this.next = null;
}
var addTwoNumbers = function(l1, l2) {
    let arr1 = []
    let arr2 = []
    let cur1 = l1
    let cur2 = l2
    while (cur1) {
        arr1.push(cur1)
        cur1 = cur1.next
    }
    while (cur2) {
        arr2.push(cur2)
        cur2 = cur2.next
    }
    let flag = arr1.length > arr2.length
    let ex = 0
    while (arr1.length || arr2.length) {
        let tmp1 = arr1.pop()
        let tmp2 = arr2.pop()
        let val = 0
        if (tmp1 && !tmp2) {
            val = ex + tmp1.val
        } else if (tmp2 && !tmp1) {
            val = ex + tmp2.val
        } else {
            val = ex + tmp1.val + tmp2.val
        }
        if (val >= 10) {
            ex = 1
            val = val - 10
        } else {
            ex = 0
        }
        if (flag) {
            tmp1.val = val
        } else {
            tmp2.val = val
        }
    }
    let ret
    if (ex > 0) {
        ret = new ListNode(ex)
    }
    if (flag) {
        if (ret) {
            ret.next = l1
        } else {
            ret = l1
        }
    } else {
        if (ret) {
            ret.next = l2
        } else {
            ret = l2
        }
    }
    return ret
};
```
