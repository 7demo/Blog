# 2-3树

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

### 删除

1, 删除叶子节点

    a, 当前节点为3-节点，则直接删除即可

    b, 当前接只有一个元素，先删除：

        a)，兄弟节点是3-节点，则将父节点移动到当前位置，再从兄弟节点中最接近当前位置的key移动到父节点

        b), 兄弟节点是2-节点，如果父节点是3-节点，从父节点降元（从3-变成2-），与兄弟节点进行合并；如果父节点是2-节点，则父兄合并进行调整。

2，删除子节点

    通过中序遍历，查找删除节点的后续节点M，把该节点的值换成M节点的值，然后删除M节点。
