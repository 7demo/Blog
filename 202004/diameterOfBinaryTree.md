# 二叉树的直径

> 何为直径，就是以某个节点左右两个支树的高度加上1（本节点的告诉）

```javaScript
var diameterOfBinaryTree = function(root) {
    let ret = 1
    function help(tree) {
        if (!tree) {
            return 0
        }
        let l = help(tree.left)
        let r = help(tree.right)
        ret = Math.max(ret, l + r + 1)

        return Math.max(l, r) + 1
    }
    help(root)
    return ret - 1
};
```
