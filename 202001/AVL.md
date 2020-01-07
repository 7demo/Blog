# 二叉树及平衡二叉树

> 子树高度差不超过1的话称为平衡二叉树。它的查找时间复杂度为O(logn)。同时二叉排序树的最大时间为O(n).

```javaScript
function Node(data, left, right) {
    this.data = data
    this.left = left
    this.right = right
}
```

## 平衡二叉树均衡

二叉树的左子节点减去右子节点的值只能为1，0或者-1，这是平衡因子。距离插入节点最近且平衡因子的绝对值大于1的节点称为最小不平衡树。

构建时，可以通过4中情况的旋转来实现平衡——LL,RR,LR,RL，即左旋、右旋、先左旋再右旋，先右旋在左旋

```javaScript
// 判断是否平衡二叉树，
function isBa(root) {
    if (!root) {
        return true
    }
    if (Math.abs(dp(root.left) - dp(root.right)) > 1) {
        return false
    }
    return isBa(root.left) && isBa(root.right)
    function dp(node) {
        if (!node) {
            return 0
        }
        let left = dp(node.left)
        let right = dp(node.right)
        return Math.max(left, right) + 1
    }
    return true
}
```

*LL* 左旋转节点N，则会把右子节点R替换N，原R的左子节点变为N的右节点，整个N变成R的左子节点。

> 当右节点的右侧高度大的时候

```javaScript
// node为旋转点，也是顶点
function LL(node) {
    let tmp = node.right
    node.right = tmp.left
    tmp.left = node
    return tmp
}
```

*RR* 右单旋子节点N，则会把左子节点R替换N, 原R的右子节点变为N的左子节点。整个N变成R的右子节点

> 当左节点的左侧高度达的时候

```javaScript
function RR(node) {
    let tmp = node.left
    node.left = tmp.right
    tmp.right = node
    return tmp
}
```

*LR* 表示先左单旋，再右单旋。

> 当左节点的右子节点的高度大的时候

```javaScript
function LR(node) {
    node.left = LL(node.left)
    return RR(node)
}
```

*RL* L则是先右单旋，再左单旋。

> 当右节点的左子节点的高度大的时候

```javaScript
function RL(node) {
    node.right = RR(node.right)
    return LL(node)
}
```

### 平衡二叉树自平衡

```javaScript
function getTreeHeight(node) {
    if (!node) {
        return 0
    }
    let leftH = getTreeHeight(node.left)
    let rightH = getTreeHeight(node.right)
    return Math.max(leftH, rightH) + 1
}
function balance(node) {
    if (node == null) {
        return node
    }
    // 大于1，则没有处于平衡状态, 左侧大，肯定第一次是右旋
    if (getTreeHeight(node.left) - getTreeHeight(node.right) > 1) {
        // 最小不平衡树是左侧 则只右单旋
        if (getTreeHeight(node.left.left) - getTreeHeight(node.left.right) > 0) {
            node = RR(node)
        } else {
            node = LR(node)
        }
    } else if (getTreeHeight(node.right) - getTreeHeight(node.left) > 1) {
        // 最小不平衡树是左侧 则只右单旋
        if (getTreeHeight(node.right.right) - getTreeHeight(node.right.left) > 0) {
            node = LL(node)
        } else {
            node = RL(node)
        }
    }
    return node
}
```

### 平衡二叉树添加

```javaScript
function Node(data, left, right) {
    this.data = data
    this.left = left
    this.right = right
}
function AVL() {
    this.root = null
}
function getTreeHeight(node) {
    if (!node) {
        return 0
    }
    let leftH = 0
    let rightH = 0
    if (node.left){
        leftH = getTreeHeight(node.left)
    }
    if (node.right) {
        rightH = getTreeHeight(node.right)
    }
    return Math.max(leftH, rightH) + 1
}
function LL(node) {
    let tmp = node.right
    node.right = tmp.left
    tmp.left = node
    return tmp
}
function RR(node) {
    let tmp = node.left
    node.left = tmp.right
    tmp.right = node
    return tmp
}
function LR(node) {
    node.left = LL(node.left)
    return RR(node)
}
function RL(node) {
    node.right = RR(node.right)
    return LL(node)
}
function balance(node) {
    if (node == null) {
        return node
    }
    node.left = balance(node.left)
    node.right = balance(node.right)
    // 大于1，则没有处于平衡状态, 左侧大，肯定第一次是右旋
    if (getTreeHeight(node.left) - getTreeHeight(node.right) > 1) {
        // 最小不平衡树是左侧 则只右单旋
        if (getTreeHeight(node.left.left) - getTreeHeight(node.left.right) > 0) {
            node = RR(node)
        } else {
            console.log('----->', JSON.stringify(node))
            node = LR(node)
        }
    } else if (getTreeHeight(node.right) - getTreeHeight(node.left) > 1) {
        // 最小不平衡树是左侧 则只右单旋
        if (getTreeHeight(node.right.right) - getTreeHeight(node.right.left) > 0) {
            node = LL(node)
        } else {
            node = RL(node)
        }
    }
    return node
}
AVL.prototype.insert = function insert(data) {
    if (!this.root) {
        this.root = new Node(data)
    } else {
        let current = this.root
        while (true) {
            // 进入左树
            if (data < current.data) {
                if (current.left == null) {
                    current.left = new Node(data)
                    break
                } else {
                    current = current.left
                }
            // 进入右树
            } else {
                if (current.right == null) {
                    current.right = new Node(data)
                    break
                } else {
                    current = current.right
                }
            }
        }
        this.root = balance(this.root)
    }
}
```

实例代码：

```javaScript
let tree = new AVL()
let arr = new Set([10,11,6,7,8,9]) // ,6,7,8,9
arr.forEach(item => {
    tree.insert(item)
})
```

### 平衡二叉树删除

删除节点时，有三种情况：

1，当前左节点没有左子树，只需要当前节点右子树覆盖当前节点

2，当前节点没有右子树，只需要当前节点左子树覆盖当前节点

3，当前左右子树都有，有两种操作：A, 当左子树高度大于右子树高度时，寻找该节点左支树最大点进行交换，B，当右子树高度大于左子树时，需要该节点右支树最小点进行交换，然后删除该节点进行再平衡

```javaScript
AVL.prototype.remove = function(data) {
    let current = this.root
    while (true) {
        // 进入左树
        if (data < current.data) {
            current = current.left
        // 进入右树
        } else if (data > current.data) {
            current = current.right
        } else if (data == current.data) {
            if (current.left == null) {
                current = current.right
                break
            } else if (current.right == null) {
                current = current.left
                break
            } else {
                // 找左子树的最大节点
                if (getTreeHeight(current.left) > getTreeHeight(current.right)) {
                    // 左分支
                    let _right = current.left.right
                    let _max = current.left.data
                    while (_right) {
                        _max = _right.data
                        _right = _right.right || null
                    }
                    current.data = _max
                    // 找右子树的最小节点
                } else {
                    // 左分支
                    let _left = current.right.left
                    let _min = current.right.data
                    while (_left) {
                        _min = _left.data
                        _left = _left.left || null
                    }
                    current.data = _min
                }
                break
            }
        }
    }
    this.root = balance(this.root)
}
```
