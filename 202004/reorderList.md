# 重排链表

> 通过快慢链表，找到中间节点。链表后半段翻转。然后依次合并链表。注意当翻转的链表合并完时，要注意设置null

```javaScript
function reverse(root) {
    if (!root) {
        return root
    }
    let cur = root
    let next = cur.next
    cur.next = null
    while (cur && next) {
        let tmp = next.next
        next.next = cur
        cur = next
        next = tmp
    }
    return cur
}
var reorderList = function(head) {
    if (!head) {
        return head
    }
    let fast = head.next
    let slow = head
    while (fast && slow) {
        slow = slow.next
        if (fast.next) {
            fast = fast.next.next
        } else {
            fast = fast.next
        }
    }

    let rev = reverse(slow)
    let ret = head
    while (ret && rev) {
        let tmp = ret.next
        let tmprev = rev.next
        ret.next = rev
        ret.next.next = tmp
        rev = tmprev
        if (ret.next) {
            ret = ret.next.next
        } else {
            ret = null
        }
    }
    if (ret) {
        ret.next = null
    }
    return head
};
```
