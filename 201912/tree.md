# 二叉树

> 最下面一层的节点层位叶节点，左侧节点都比父节点小，右侧节点都比父节点大

## 二叉树的实现

> 此处实现的是二叉排序树/二叉搜索树/二叉查找树。

```javaScript
function Node(data, left, right) {
        this.data = data
        this.left = left
        this.right = right
        this.show = show
    }
    function show() {
        return this.data
    }
    function BIT() {
        this.root = null
        this.insert = insert
    }
    function insert(data) {
        let node = new Node(data)
        if (!this.root) {
            this.root = node
        } else {
            let current = this.root
            let parent;
            while(true) {
                parent = current
                if (data < current.data) {
                    current = current.left
                    if (current == null) {
                        parent.left = node
                        break
                    }
                } else {
                    current = current.right
                    if (current == null) {
                        parent.right = node
                        break
                    }
                }
            }
        }
    }
```

#### 满二叉树

二叉树的高度为n，节点数为2^n - 1(一个二叉树，节点最多也只能为2^n - 1，第i行，节点数最多为2^(i-1))

#### 完全二叉树

完全二叉树是基于满二叉树，如果二叉树的深度为n， 那么低n-1层是一个满二叉树，第n层的节点都在最左侧。

#### 平衡二叉树（AVL）

子树高度差不超过1的话称为平衡二叉树。它的查找时间复杂度为O(logn)。同时二叉排序树的最大时间为O(n).

## 查找

我们先用用以上代码生成一个二叉树：

```javaScript
const keys = [8,3,10,1,6,14,4,7,13]
let tree = new BIT()
keys.map(item => {
    tree.insert(item)
})
```

#### 中序遍历

```javaScript
function cb(arg) {
    console.log(arg)
}
function findMid(tree, cb) {
    if (tree != null) {
        findMid(tree.left, cb)
        cb(tree.data)
        findMid(tree.right, cb)
    }
}
//  1 3 4 6 7 。。。 14
```

整个结果是以从小到大依次显示。

那么前序遍历就是，先根再左再右；后序排列是先左再右再跟。

其实，深度优先搜索算法则是前序遍历。

#### 查找BIS最小值和查找BIS最大值

查找BIS最小值： 查找二叉树最左侧的没有子节点的节点

查找BIS最大值：查找二叉树最右侧的没有子节点的节点


#### 广度优先搜索算法

```javaScript
function bf(tree) {
    let arr = []
    let ret = []
    if (tree) {
        arr.push(tree)
        while(arr.length) {
            let node = arr.shift()
            ret.push(node.data)
            node.left && arr.push(node.left)
            node.right && arr.push(node.right)
        }
    }
    console.log(ret)
    return ret
}
```

#### 深度优先搜索算法

```javaScript
function bf(tree) {
    let arr = []
    let ret = []
    if (tree) {
        arr.push(tree)
        while(arr.length) {
            let node = arr.pop()
            ret.push(node.data)
            node.left && arr.push(node.left)
            node.right && arr.push(node.right)
        }
    }
    console.log(ret)
    return ret
}
```

#### 根据前序求二叉树

#### 翻转二叉树

```javaScript
function tf(tree) {
    if (tree != null) {
        tf(tree.left)
        tf(tree.right)
        let tmp = tree.right
        tree.right = tree.left
        tree.left = tmp
    }
}
```

#### 平衡二叉树均衡

二叉树的左子节点减去右子节点的值只能为1，0或者-1，这是平衡因子。距离插入节点最近且平衡因子的绝对值大于1的节点称为最小不平衡树。

构建时，可以通过4中情况的旋转来实现平衡——LL,RR,LR,RL.

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

*LL* 左旋转节点N，则会把右子节点R替换N，原R的左子节点变为N的右节点，整个N变成R的左子节点

```javaScript
function LL(node) {
    let tmp = node.right
    node.right = tmp.left
    tmp.left = node
    return tmp
}
```

*RR* 右单旋子节点N，则会把左子节点R替换N, 原R的右子节点变为N的左子节点。整个N变成R的右子节点

```javaScript
function RR(node) {
    let tmp = node.left
    node.left = tmp.right
    tmp.right = node
    return tmp
}
```

*LR* 表示先左单旋，再右单旋。RL则是先右单旋，再左单旋。

至于适合哪种旋转，简单来说，则是哪侧支点深度小，则使用哪侧转。比如左侧树深度大，则使用右转RR。

```javaScript
function balance(node) {
    if (node == null) {
        return node
    }
    // 大于1，则没有处于平衡状态, 左侧大，肯定第一次是右旋
    if (getTreeHeight(node.left) - getTreeHeight(node.right) > 1) {
        // 最小不平衡树是左侧 则只右单旋
        if (getTreeHeight(node.left.left) - getTreeHeight(node.left.right) > 1) {
            node = RR(node)
        } else {
            node = RL(node)
        }
    } else if (getTreeHeight(node.right) - getTreeHeight(node.left) > 1) {
        // 最小不平衡树是左侧 则只右单旋
        if (getTreeHeight(node.right.right) - getTreeHeight(node.right.left) > 1) {
            node = LL(node)
        } else {
            node = LR(node)
        }
    }
    return node
}
```

## B树/2-3树

> 正如以上，平衡二叉树（BST）查找最极端情况变成了链表，时间复杂度为O(n)。那为什么会产生了极端的情况？

主要原因是： *插入节点时，在无法预知未来节点时，只能机械的比较节点的值进行左插入或者右插入，再进行平衡*

如果我们插入节点时，能够为再次插入节点留有余地呢？就引申出2-3树了。最坏情况下2-3树查找效率为O(Logn), 最好就是所有节点都是3节点，效率为(0.63logn).

2-3树就是每个节点可以有1个或者2个key，当只有1个key时，可以没有子节点或者有两个子节点（不能只有一个节点），左侧节点小于key，右侧节点大于key；当有2个key时，可以没有子节点或者右二/三个子节点，左节点小于两个key，中间节点位于两个key中间，右侧节点大于两个key。

基于以上定义，在插入节点时，不向下生成左侧或者右侧节点，而把插入值放入该节点，如果该树节点的key大于2时，则把三个key中大小位于中间的值向上节点插入，当前节点则分为左右节点。

```javaScript
// 节点具有两个节点三个分支，左节点小于右节点
function Node(key) {
    this.key1 = key
    this.key2 = null
    this.left = null
    this.mid = null
    this.right = null
    this.isHasNoChild = isHasNoChild
    this.isFull = isFull
    this.hasKey = hasKey
    this.getChild = getChild
}
function isHasNoChild() {
    return this.left == null && this.mid == null && this.right == null
}
function isFull() {
    return this.key2 != null
}
function hasKey(key) {
    if (this.key1 == key || (this.key2 == key && this.key2 != null)) {
        return true
    } else {
        return false
    }
}
function getChild(key) {
    if (key < this.key1) {
        return this.left
    } else if (this.key2 == null || key < this.key2) {
        return this.mid
    } else {
        this.right
    }
}

function tsTree() {
    let root = null
    this.put = function put(key) {
        if (this.root == null) {
            this.root = new Node(key)
        } else {
            // 此处捕获增加节点时，如果遇到拆分冒泡的情况——会返回拆分节点的中间大小的key与右key的节点
            // pkey表示可以变成父节点的key
            // pRef表示最大key的新node
            let [pkey, pRef] = this._put(this.root, key)
            if (pRef != null) {
                let newNode = new Node(pkey)
                newNode.left = this.root
                newNode.mid = pRef
                this.root = newNode
            }
        }
    }
    this._put = function _put(node, key) {
        // 如果节点以及有该key
        if (node.hasKey(key)) {
            return [null, null]
        // 如果该节点没有子节点，最直接添加
        } else if (node.isHasNoChild()) {
            return this._addNode(node, key, null)
        // 否则遍历该节点的子节点进行添加
        } else {
            let child = node.getChild(key)
            // 遍历时遇到拆分节点时，则从该节点进行处理
            let [pkey, pRef] = this._put(child, key)
            if (pkey != null) {
                return [null, null]
            } else {
                return this._addNode(node, pkey, pRef)
            }
        }
    }
    this._addNode = function _addNode(node, key, pRef) {
        if (node.isFull()) {
            return this._splitNode(node, key, pRef)
        } else {
            if (key < node.key1) {
                node.key2 = node.key1
                node.key1 = key
                if (pRef != null) {
                    node.right = node.mid
                    node.mid = pRef
                }
            } else {
                node.key2 = key
                if (pRef != null) {
                    node.right = pRef
                }
            }
            return [null, null]
        }
    }
    this._splitNode = function _splitNode(node, key, pRef) {
        let newNode = new Node(null)
        let pkey = null
        if (key < node.key1) {
            pkey = node.key1
            node.key1 = key
            newNode.key1 = node.key2
            if (pRef != null) {
                newNode.left = node.mid
                newNode.mid = node.right
                node.mid = pRef
            }
        } else if (key < node.key2) {
            pkey = key
            newNode.key1 = node.key2
            if (pRef != null) {
                newNode.left = pRef
                newNode.mid = node.right
            }
        } else {
            pkey = node.key2
            newNode.key1 = key
            if (pRef != null) {
                newNode.left = node.right
                newNode.mid = pRef
            }
        }
        node.key2 = null
        return [pkey, newNode]
    }
}
```

参考：[算法原理系列：2-3查找树](https://blog.csdn.net/u014688145/article/details/67636509)

## 红黑树

> 平衡二叉树的基本效率是o(logn)，但是删除与插入因为额外配平是浪费一些效率。所以使用的二叉树都是红黑树。红黑树不是绝对平衡，而是相对平衡。

### 特性

1，根节点是黑的。

2，节点是黑或者红的

3，每个叶子节点都是黑的`null`空节点

4，每个红色节点的两个子节点都是黑色的。

5，任一节点到其叶节点都包含相同的黑色节点。
