# LRU缓存机制

> 要求缓存大小有时间、操作时间为0（1），能自动删除最久值

关键是更新值后如何实现O(1)，这里借助双向链表，把已经存在的值保存hash的同时做一个链表，存头与尾两个指针。每次操作更新指针。

```javaScript
function ListNode(val) {
    this.val = val;
    this.pre = null
    this.next = null;
}

var LRUCache = function(capacity) {
    this.len = capacity // 长度
    this.size = 0 // 代表当前长度
    this.keyList = null
    this.tail = null // 链表尾部
    this.nodeObj = {} // key所对应的链表数据
    this.obj = {} // 存放key value
};

/**
    * @param {number} key
    * @return {number}
    */
LRUCache.prototype.get = function(key) {
    if (this.obj[key]) {
        let node = this.nodeObj[key]
        let _nodepre = node.pre
        let _nodenext = node.next
        // 如果是当中节点
        if (_nodepre && _nodenext) {
            _nodepre.next = _nodenext
            _nodenext.pre = _nodepre
            this.tail.next = node
            node.next = null
            node.pre = this.tail
            this.tail = node
        // 表明是链表顶部
        } else if (!_nodepre && _nodenext) {
            this.keyList = this.keyList.next
            this.keyList.pre = null
            this.tail.next = node
            node.next = null
            node.pre = this.tail
            this.tail = node
        }
        return this.obj[key]
    } else {
        return -1
    }
};

/**
    * @param {number} key
    * @param {number} value
    * @return {void}
    */
LRUCache.prototype.put = function(key, value) {
    // 如果值已经存在，更新值，更新链表尾
    if (this.obj[key]) {
        this.obj[key] = value
        let node = this.nodeObj[key]
        let _nodepre = node.pre
        let _nodenext = node.next
        // 如果是当中节点
        if (_nodepre && _nodenext) {
            _nodepre.next = _nodenext
            _nodenext.pre = _nodepre
            this.tail.next = node
            node.next = null
            node.pre = this.tail
            this.tail = node
        // 表明是链表顶部
        } else if (!_nodepre && _nodenext) {
            this.keyList = this.keyList.next
            this.keyList.pre = null
            this.tail.next = node
            node.next = null
            node.pre = this.tail
            this.tail = node
        }
        // 最后是链表表尾部，则不处理
    // 如果值不存
    } else {
        this.obj[key] = value
        let len = this.size
        let node = new ListNode(key)
        // 可以直接新增
        if (this.len > len) {
            // 非第一次新增
            if (this.keyList) {
                this.tail.next = node
                node.pre = this.tail
                this.tail = node
            } else {
                this.keyList = node
                this.tail = node
            }
            this.size = this.size + 1
        // 则需要新增后删除顶部
        } else {
            this.tail.next = node
            node.pre = this.tail
            this.tail = node
            let delNodeVal = this.keyList.val
            delete this.obj[delNodeVal]
            delete this.nodeObj[delNodeVal]
            this.keyList = this.keyList.next
            this.keyList.pre = null
        }
        this.nodeObj[key] = node
    }
};
```

