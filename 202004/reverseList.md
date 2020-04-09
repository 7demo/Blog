# 翻转链表

### 翻转完整链表

> 递归解法。时间复杂度为O(n), 但是需要额外O(n)的空间。

```javaScript
var reverse = function(head) {
    if (!head || !head.next) {
        return head
    }
    let re = reverse(head.next)
    head.next.next = head
    head.next = null
    return re
}
```

> 迭代解法。时间复杂度为O(n)， 空间为常量。

```javaScript
var reverse = function(head) {
    if (!head || !head.next) {
        return head
    }
    let cur = null
    let next = head
    while (next) {
        let tmp = next.next
        next.next = cur
        cur = next
        next = tmp
    }
    return cur
}
```

### 翻转链表前n节点

> 递归

```javaScript
var reverseN = function(head, n) {
    // 存放n节点后的其他节点
    let tail = null
    function core(head, n) {
        if (n==1) {
            tail = head.next
            return head
        }
        if (!head || !head.next) {
            return head
        }
        let re = core(head.next, n-1)
        head.next.next = head
        // 虽然每次递归时都会被赋值，tail，但是到上一级会被head.next.next纠正
        head.next = tail
        return re
    }
    return core(head, n)
}
```

> 迭代

```javaScript
var reverseN = function(head, n) {
    let cur = null
    let next = head
    // 用于拼接最后断掉的next
    let headEx = head
    // 实时指针，用于最终返回
    let ret = null
    while (next && n) {
        if (n == 0) {
            break
        }
        let tmp = next.next
        next.next = cur
        ret = next
        cur = next
        next = tmp
        n--
    }
    headEx.next = next
    return ret
}
```

### 翻转链表m~n的节点

```javaScript
var reverseBetween = function(head, m, n) {
    if (m == 1) {
        return reverseN(head, n)
    }
    head.next = reverseBetween(head.next, m-1, n-1)
    return head
};
```
