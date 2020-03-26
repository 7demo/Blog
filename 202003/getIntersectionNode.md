# 相交链表

> 其实是判断节点是否相等，不是值是否相等

> 思路一：遍历heada, 增加属性，再遍历headB, 如果遇到第一个改变的值就是。

> 思路二：双指针，heada 的尾接headb的头，第一个相等的就是

```javaScript
var getIntersectionNode = function(headA, headB) {
   let cA = headA
    let cB = headB
    while (true) {
        if (cA == cB) {
            return cA
        }
        cA = cA ? cA.next : headB
        cB = cB ? cB.next : headA
    }
};
```
