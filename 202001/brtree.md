# 红黑树

> 为了追求效率，从二叉树到了2-3树，但是因为有3树的存在，操作不方便，因此把有两个key的节点拆分，变成红色节点。就是红黑树。

### 特性

1，根节点是黑的。

2，节点是黑或者红的

3，每个叶子节点都是黑的`null`空节点

4，每个红色节点的两个子节点都是黑色的。

5，任一节点到其叶节点都包含相同的黑色节点。

6, 最短路径时，该路径的节点必为黑色节点构成; 没有任何一条路径比其他路径长两倍。

7，最长路径时，该路径的节点比为红黑节点间隔构成


### 旋转

旋转规则与平衡二叉树一样。

### 时间复杂度

红黑树的时间复杂度为O(logn)

1, 含有n个节点的红黑树的高度至多为2log(n+1)

2, 高度为h的红黑树，节点数至少为2^bh(x) - 1


### 实现

1，红黑树为空树，则根节点就是插入节点，改变该节点颜色为黑色

2，如果插入节点已经存在，那么只需要更新插入节点的颜色

3，如果父节点直接为黑色，那么直接插入

4，如果父节点为红色，叔叔节点为红色时:

    a，需要把父节点、叔叔节点调整成黑色，爷爷节点调整成红色。并且要基于爷爷节点网上进行颜色检查。
<!--
    c，（此时是检查状态，并非第一次插入状态）如果叔叔节点为黑色，插入节点是父节点的右孩子，切父节点位于爷爷节点的左树，则对父节点进行左旋（此时插入节点，不是真正意义的插入节点，而是在此节点上插入节点后调整后的节点）。 -->

5，如果父节点为红色，不存在叔叔节点或者是叔叔节点为黑色时：

    a，如果插入节点为右孩子，那么父节点为爷爷节点的左孩子，那么以父节点进行左旋；如果插入节点为左孩子，那么父节点为右孩子，那么右旋父节点。

    b, 如果插入节点为左孩子，那么父节点变黑色，爷爷节点变红。如果父节点是做左孩子，则爷爷节点进行右旋，否则爷爷节点左旋。

    ---

    以上旋转后，则需要把父节点设置为黑色，爷爷节点设置为红色。因为已旋转，此时父节点其实是爷爷节点的父节点，所以不要继续再查找。


参考：[数据结构（十三）之红黑树](https://www.jianshu.com/p/00aae4f4d672)

```javaScript
function Node(value) {
	this.value = value
	this.color = 'red' // 结点的颜色默认为红色
	this.parent = null
	this.left = null
	this.right = null
}

function RedBlackTree() {
 	this.root = null
}

RedBlackTree.prototype.insert = function (node) {

	// 以二叉搜索树的方式插入结点
	// 如果根结点不存在，则结点作为根结点
	// 如果结点的值小于node，且结点的右子结点不存在，跳出循环
	// 如果结点的值大于等于node，且结点的左子结点不存在，跳出循环
	if (!this.root) {
		this.root = node
	} else {
		let current = this.root
		while (true) {
			if (node.value < current.value) {
				if (current.left) {
					current = current.left
				} else {
					node.parent = current
					current.left = node
					break
				}
			} else if (node.value < current.value) {
				if (current.right) {
					current = current.right
				} else {
					node.parent = current
					current.right = node
					break
				}
			}
		}
	}
	// 判断情形
	this._fixTree(node)
	return this

}

RedBlackTree.prototype._fixTree = function (node) {
 	// 当node.parent不存在时，即为情形1，跳出循环
 	// 当node.parent.color === 'black'时，即为情形2，跳出循环
 	while (node.parent && node.parent.color !== 'black') {
 		let father = node.parent
		let grand = father.parent
		let uncle = grand[grand.left === father ? 'right' : 'left']
		// 叔叔节点不为黑色，也可认为是黑色
		if (!uncle || uncle.color === 'black') {
			// 判断父节点与爷爷节点在其父节点的左右支树
			let directionFromFatherToNode = father.left === node ? 'left' : 'right'
			let directionFromGrandToFather = grand.left === father ? 'left' : 'right'
			// 父节点与爷爷节点在同一侧
			if (directionFromFatherToNode === directionFromGrandToFather) {
				this._rotate(father)
				// 变色
				father.color = 'black'
				grand.color = 'red'
			} else {
				this._rotate(node)
				this._rotate(node)
				// 变色
				node.color = 'black'
				grand.color = 'red'
			}
			// 完成插入，跳出循环
			break
		// 父节点与叔叔节点都是红色，只需要简单的翻转颜色即可，然后继续向上查找
		} else {
			grand.color = 'red'
			father.color = 'black'
			uncle.color = 'black'
			// 将grand设为新的node
			node = grand
		}
 	}
 	if (!node.parent) {
		node.color = 'black'
		this.root = node
 	}

}

RedBlackTree.prototype.LL = function(node) {
	let tmp = node.right
	node.right = tmp.left
	tmp.left = node
	return tmp
}

RedBlackTree.prototype.RR = function(node) {
	let tmp = node.left
	node.left = tmp.right
	tmp.right = node
	return tmp
}

RedBlackTree.prototype._rotate = function (node) {
	// 旋转 node 和 node.parent
	let parent = node.parent
	// 如果位于父节点的右分支---左旋
	if (parent.right === node) {
		// 如果存在爷爷节点，则把子节点放到父节点的位置上
		if (parent.parent) {
			parent.parent[parent.parent.left === parent ? 'left' : 'right'] = node
		}
		// 同时把子节点的父节点指向爷爷节点
		node.parent = parent.parent
		// 把node的左节点挂到parent的右节点，
		if (node.left) {
			node.left.parent = parent
		}
		parent.right = node.left
		node.left = parent
		parent.parent = node
	// 如果位于左分支，右旋
	} else {
		if (parent.parent) {
			parent.parent[parent.parent.left === parent ? 'left' : 'right'] = node
		}
		node.parent = parent.parent
		if (node.right) {
			node.right.parent = parent
		}
		parent.left = node.right
		node.right = parent
		parent.parent = node
	}
}
```

### 删除

1，删除的节点无子节点时：

	a, 该节点为黑色时，删除后要进行平衡操作

	b, 该节点为红色，则直接删除即可。

2，删除的节点有一个节点时，该节点只能为黑色，子节点为红色。则删除节点，把子节点连接到父节点，子节点涂黑。

3，删除的节点右两个节点时，则用后序节与删除节点的值进行互换，此时，原后序节点变成了情况1或者情况2.

### 删除再平衡


1, 如果平衡节点为根节点，则不需要操作

2，如果平衡节点的兄弟节点为黑色：

	a，兄弟节点的子节点都为黑色，则把兄弟节点涂红，再看父节点：

		1)，如果父节点为黑色，则把父节点作为新的平衡节点进行处理

		2), 如果父节点为红色，则把父节点涂黑即可。

	b, 兄弟节点的子节点不全黑：

		1)，兄弟节点为黑，兄弟节点位于左树，兄弟节点的左子节点为红，那么父节点进行右旋，交换父节点与兄弟节点的颜色，把兄弟节点的左子节点涂黑

		2)，兄弟节点为黑，兄弟节点位于右树，兄弟节点的右子节点为黑，那么热父节点进行左旋，交换父兄节点的颜色，把兄弟节点的右子节点涂黑

		3)，兄弟节点为黑色，兄弟节点位于左树，兄弟节点的右子树为红，那么兄弟节点进行左旋，交换兄弟节点与兄弟节点右子节点的颜色。以下用2-b-1

		4)，兄弟节点为黑色，兄弟节点位于右树，兄弟节点的左子树为红，那么兄弟节点进行右旋，交换兄弟节点与兄弟节点做子树节点的颜色，以下用2-b-2。

3，如果平衡节点的兄弟节点为红色：

	a)，兄弟节点在左，那么父节点进行右旋，交换父兄节点颜色，以下用2情况解决。

	b)，兄弟节点在右，那么父节点进行左旋，交换父兄节点颜色，以下用2情况解决。

参考：[彻底理解红黑树（三）之 删除](https://www.jianshu.com/p/84416644c080)


