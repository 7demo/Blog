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

## 翻转二叉树

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

