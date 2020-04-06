#链表排序

> 利用快慢针，切分两个链表进行排序

```javaScript
var sortList = function(head) {
    return merge(head)
};
function merge(head) {
    if (!head || !head.next) {
        return head
    }
    let fast = head.next
    let slow = head
    while (fast && fast.next) {
        slow = slow.next
        fast = fast.next.next
    }
    let tail = slow.next
    slow.next = null
    return mergeSort(merge(head), merge(tail))
}
function ListNode(val) {
    this.val = val;
    this.next = null;
}
function mergeSort(l, r) {
    let list
    let cur
    while (l || r) {
        let node = new ListNode()
        if (!r && l) {
            node.val = l.val
            l = l.next
        } else if (!l && r) {
            node.val = r.val
            r = r.next
        } else {
            if (l.val < r.val) {
                node.val = l.val
                l = l.next
            } else {
                node.val = r.val
                r = r.next
            }
        }
        if (!list) {
            list = node
        } else {
            cur.next = node
        }
        cur = node
    }
    return list
}
```
