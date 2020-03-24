# 二叉树的右视图

> 关键时，每层如果右树不存在的话，是要看到左树的。

```javaScript
var rightSideView = function(root) {
    if (!root) {
        return []
    }
    let ret = []
    let q = [root]
    while (q.length) {
        ret.push(q[0].val)
        let len = q.length
        while (len--) {
            let node = q.shift()
            if (node.right) {
                q.push(node.right)
            }
            if (node.left) {
                q.push(node.left)
            }
        }
    }
    return ret
};
```
