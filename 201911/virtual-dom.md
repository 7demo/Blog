# Vue中的Virtual-dom

## Virtual-dom

首先，我们要知道`dom`操作是昂贵的，主要是源自两个方面：

1，真实元素有了太多的属性与方法，这势必比较臃肿，会造成额外的性能浪费。

2，dom操作是多次进行的，不断的进行重绘与重排，就会产生重复的dom操作。

因此，就产生了`Virtual-dom`，所有的元素通过计算，用js对象表达，然后再一次性的渲染出来。势必会大大的节省渲染时间。这里需要知道的是，`虚拟dom`其实就是js对象，一般还有`tag`、`props`、`children`这几个属性，分别表示标签名、属性、子元素。

我们首先声明一个方法，来产生虚拟dom。

```javascript
let index = 1
function createData() {
    if (index === 11) {
        index = 1
    }
    let _data = {
        tag: 'div',
        props: {},
        children: [
            'hello, word',
            {
                tag: 'ul',
                props: {},
                children: []
            }
        ]
    }
    for (let i = 0; i < index; i++) {
        _data.children[1].children.push({
            tag: 'li',
            props: {
                id: i,
                class: `id-${i}`
            },
            children: [`第${i}个元素`]
        })
    }
    index ++
    return _data
}
```

我们在根据虚拟dom来实现一个创建dom的方法：

```javascript
function createEle(data) {
    // 如果是文本对象
    if (typeof data === 'string' || typeof data === 'number') {
        return document.createTextNode(data)
    }
    // 如果元素对象
    let parentEle = document.createElement(data.tag)
    for (let k in data.props) {
        parentEle.setAttribute(k, data.props[k])
    }

    data.children.forEach(item => {
        parentEle.appendChild(createEle(item))
    })

    return parentEle
}
```

那么我们就可以模拟虚拟dom来渲染的过程了。

```javascript
let preVdom
setInterval(() => {
    let _data = createData()
    let _ele = createEle(_data)
    if (preVdom) {
        document.body.replaceChild(_ele, preVdom)
        preVdom = document.body.children[document.body.children.length - 1]
    } else {
        preVdom = document.body.appendChild(_ele)
    }
}, 1000)
```
这样，页面就会不断循环显示列表。并且这个列表是单次操作创建插入页面的，而不是一次次创建一条数据插入的。从理论上讲，性能的确会更加好些。

这个地方操作是每次替换整个页面内容，而不是变动只需要变化的部分，这不是我们的目标。我们的目标是要实现只更新变化的部分。那我们需要来对比新旧虚拟dom，找出变化的部分。

首先，我们先设定，元素的变更有四种类型，熟悉有两种类型：

```javascript
const nodePatchTypes = {
    CREATE: 'create node', // 创建节点
    REMOVE: 'remove node', // 移除节点
    REPLACE: 'replace node', // 替换节点
    UPDATE: 'update node' // 更新节点
}

const propPatchTypes = {
    REMOVE: 'remove prop', // 移除属性
    UPDATE: 'update prop' // 更新属性
}
```

然后比较新旧dom(我们只比较同层级dom)

```javascript
function diff(oldVDom, newVDom) {
    // 新增
    if (oldVDom === undefined) {
        return {
            type: nodePatchTypes.CREATE,
            vdom: newVDom
        }
    }
    // 删除
    if (newVDom === undefined) {
        return {
            type: nodePatchTypes.REMOVE,
            vdom: oldVDom
        }
    }
    // 替换—— 节点类型不一致，都是字符串或者数字时值不一样，都是对象时标签不一致
    if (typeof oldVDom !== typeof newVDom || (typeof oldVDom === 'string' || typeof oldVDom === 'number') && oldVDom !== newVDom || oldVDom.tag !== newVDom.tag) {
        return {
            type: nodePatchTypes.REPLACE,
            vdom: newVDom
        }
    }
    // 更新节点
    if (oldVDom.tag) {
        let propsDiff = diffProps(oldVDom, newVDom)
        let childrenDiff = diffChildren(oldVDom, newVDom)
        if (propsDiff.length || childrenDiff.some(item => item !== undefined)) {
            return {
                type: nodePatchTypes.UPDATE,
                props: propsDiff,
                children: childrenDiff
            }
        }
    }
}
// 比较属性
function diffProps(oldVDom, newVDom) {
    let patchs = []
    let allProps = {...oldVDom.props, ...newVDom.props}
    Object.keys(allProps).forEach(key => {
        let oldVal = oldVDom.props[key]
        let newVal = newVDom.props[key]
        // 删除
        if (newVal === undefined) {
            patchs.push({
                type: propPatchTypes.REMOVE,
                key
            })
        } else if (oldVal === undefined || oldVal !== newVal) {
            patchs.push({
                type: propPatchTypes.UPDATE,
                key,
                value: newVal
            })
        }
    })
    return patchs
}
// 比较子节点
function diffChildren(oldVDom, newVDom) {
    let patchs = []
    let len = Math.max(oldVDom.children.length, newVDom.children.length)
    for (let i = 0; i < len; i++) {
        let _pathch = diff(oldVDom.children[i], newVDom.children[i])
        patchs.push(_pathch)
    }
    return patchs
}
```

然后我们创建两个虚拟dom进行比较：

```javascript
let data1 = createData()
let data2 = createData()
diff(data1, data2)
```

输出为：

```
{
	"type": "update node",
	"props": [],
	"children": [null, {
		"type": "update node",
		"props": [],
		"children": [null, {
			"type": "create node",
			"vdom": {
				"tag": "li",
				"props": {
					"id": 1,
					"class": "id-1"
				},
				"children": ["第1个元素"]
			}
		}]
	}]
}
```

得到新旧虚拟dom比较后，其中增加了一个li的节点。下面我们就根据差异做渲染。

```javascript
// index 表示将要更换元素是父元素的第几个
// 最顶层parent 则是vue挂在的div，会永远不会动
function patch(parent, patchObj, index = 0) {
    if (!patchObj || !parent) {
        return
    }
    if (patchObj.type === nodePatchTypes.CREATE) {
        return parent.appendChild(createEle(patchObj.vdom))
    }
    let element = parent.childNodes[index]

    if (patchObj.type === nodePatchTypes.REMOVE) {
        return parent.removeChild(element)
    }
    if (patchObj.type === nodePatchTypes.REPLACE) {
        return parent.replaceChild(element, createEle(patchObj.vdom))
    }
    if (patchObj.type === nodePatchTypes.UPDATE) {
        let {props, children} = patchObj
        patchProps(element, props)
        let len = children.length
        while (len) {
            len --
            patch(element, children[len], len)
        }
    }

}

function patchProps(element, props) {
    if (!props) {
        return
    }
    props.forEach(patchObj => {
        if (patchObj.type === propPatchTypes.UPDATE) {
            element.setAttribute(patchObj.key, patchObj.value)
        }
        if (patchObj.type === propPatchTypes.REMOVE) {
            element.removeAttribute(patchObj.key)
        }
    })
}
```
然后我们做个定时器，每一秒触发一次dom更新。

```
let oldV
setInterval(() => {
    let _data = createData()
    let _patch = diff(oldV, _data)
    // console.log('11111======111111',_patch)
    patch(document.querySelector('#main'), _patch)
    oldV = _data
}, 1000)
```

以上代码有个非必要操作，就是先比较出差异，再根据差异patch。其实这一步可以在比较出差异的时候直接更新dom。另外也需要实现新的虚拟dom直接与元素对比，而不是需要暂存旧的dom。


```
// 直接比较 newdom 与现实节点
function diff(newVDom, parent, index = 0) {
    // 现在的子元素
    let element = parent.childNodes[index]
    // 旧子元素不存在，则表示新增
    if (element === undefined) {
        return parent.appendChild(createEle(newVDom))
    }
    // 删除
    if (newVDom === undefined) {
        return parent.removeChild(element)
    }
    // 替换—— 节点类型不一致，都是字符串或者数字时值不一样，都是对象时标签不一致
    if (!isSameType(newVDom, element)) {
        return parent.replaceChild(element, createEle(newVDom))
    }
    // 更新节点
    if (element.nodeType === Node.ELEMENT_NODE) {
        diffProps(newVDom, element)
        diffChildren(newVDom, element)
    }
}

function isSameType(newVDom, element) {
    let eleType = element.nodeType
    let vdomType = typeof newVDom
    if (eleType === Node.TEXT_NODE && (vdomType === 'string' || vdomType === 'number') && element.nodeValue == newVDom) {
        return true
    }
    if (eleType === Node.ELEMENT_NODE && element.tagName.toLowerCase() == newVDom.tag) {
        return true
    }
    return false
}

function diffProps(newVDom, parent) {
    let oldProps = {...parent.__props_}
    let allProps = {...oldProps, ...newVDom.props}
    Object.keys(allProps).forEach(key => {
        let oldVal = oldProps[key]
        let newVal = newVDom.props[key]
        // 删除
        if (newVal === undefined) {
            parent.removeAttribute(key)
        } else if (oldVal === undefined || oldVal !== newVal) {
            parent.setAttribute(key, newVal)
        }
    })
    parent.__props_ = newVDom.props
}

function diffChildren(newVDom, parent) {
    let len = Math.max(parent.childNodes.length, newVDom.children.length)
    while (len) {
        len--
        diff(newVDom.children[len], parent, len)
    }
}
```

如果每条数据中有key呢？

首先，每次比较时，可以把当前子元素分为两类：一是带有key，一个是不带key。

然后，遍历虚拟dom，如果虚拟dom也有key，则找到现子元素带有同样的key进行比较，如果需要不同则处理，如果相同则直接移动位置。（这也是diff函数改造的原因，在进行对比的时候，同时标明是否需要更新）；如果虚拟dom没有key，则找现在有子元素中类似的，如果有类似元素，则进行比较是否进行处理。

最后，再第二部处理时，因为是对照虚拟dom进行处理的，所以先后处理的顺序就是页面最后显示的顺序。最后把现有元素没有用到的进行移除即刻。

```javascript
// 新增、删除、替换是直接表示改元素不需要更新，因为已处理完毕。如果是更新，则监测是否复用，此时就存在一种情况，就是父元素被复用，但是这个父元素的子元素是更改的，达到最小化的操作dom。
function diff(dom, newVDom, parent) {
    // 现在的子元素
    // 旧子元素不存在，则表示新增
    if (dom === undefined) {
        parent.appendChild(createEle(newVDom))
        return false
    }
    // 删除
    if (newVDom === undefined) {
        parent.removeChild(dom)
        return false
    }
    // 替换—— 节点类型不一致，都是字符串或者数字时值不一样，都是对象时标签不一致
    if (!isSameType(newVDom, dom)) {
        parent.replaceChild(dom, createEle(newVDom))
        return false
    }
    // 更新节点
    if (dom.nodeType === Node.ELEMENT_NODE) {
        diffProps(newVDom, dom)
        diffChildren(newVDom, dom)
    }
    return true
}
// 判断是否同一类型
function isSameType(newVDom, element) {
    let eleType = element.nodeType
    let vdomType = typeof newVDom
    if (eleType === Node.TEXT_NODE && (vdomType === 'string' || vdomType === 'number') && element.nodeValue == newVDom) {
        return true
    }
    if (eleType === Node.ELEMENT_NODE && element.tagName.toLowerCase() == newVDom.tag) {
        return true
    }
    return false
}
// 先分为两个部分：有key  无key
function diffChildren(newVDom, parent) {
    // 存放带有key元素
    let nodesWithKey = {}
    let nodesWithKeyCount = 0
    // 存放没有key元素的
    let nodesWithoutKey = []

    let childNodes = parent.childNodes
    let nodeLength = childNodes.length

    let vChildren = newVDom.children
    let vChildrenLen = vChildren.length

    // 讲现有子元素分为两组
    for (let i = 0; i < nodeLength; i++) {
        let child = childNodes[i]
        let props = child.__props_
        if (props !== undefined && props.key !== undefined) {
            nodesWithKey[props.key] = child
            nodesWithKeyCount++
        } else {
            nodesWithoutKey.push(child)
        }
    }
    // 遍历虚拟dom
    for (let i = 0; i < vChildrenLen; i++) {
        let vchild = vChildren[i]
        let vProps = vchild.props

        let dom
        let vkey = vProps !== undefined ? vProps.key : undefined
        if (vkey !== undefined) {
            if (nodesWithKeyCount && nodesWithKey[vkey] !== undefined) {
                dom = nodesWithKey[vkey]
                nodesWithKey[vkey] = undefined
                nodesWithKeyCount--
            }
        } else if(nodesWithoutKey.length) {
            for (let j = 0; j < nodesWithoutKey.length; j++) {
                let node = nodesWithoutKey[j]
                if (node !== undefined && isSameType(vchild, node)) {
                    dom = node
                    delete nodesWithoutKey[j]
                    break
                }
            }
        }
        let isUpdate = diff(dom, vchild, parent)
        if (isUpdate) {
            let originChild = childNodes[i]
            if (originChild !== dom) {
                parent.insertBefore(dom, originChild)
            }
        }
    }
    if (nodesWithKeyCount) {
        for (let key in nodesWithKey) {
            let node = nodesWithKey[key]
            if (node !== undefined) {
                node.parentNode.removeChild(node)
            }
        }
    }
    while (nodesWithoutKey.length) {
        let node = nodesWithoutKey.pop()
        if (node !== undefined) {
            node.parentNode.removeChild(node)
        }
    }
}
```

以上参考：(https://segmentfault.com/a/1190000016129036)[https://segmentfault.com/a/1190000016129036]

## Vue中的Virtual-Dom

`Vue`的学习可以参考(https://github.com/DDFE/DDFE-blog/issues/18)[https://github.com/DDFE/DDFE-blog/issues/18]
