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
