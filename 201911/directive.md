# directive

## 源码探究

同`component`一样，`Vue`扩展了一个`Vue.directive`一个静态方法，用来注册指令。

```javaScript
Vue[type] = function (
    id: string,
    definition: Function | Object
): Function | Object | void {
    // 可以看到如果directive第二个参数直接是一个function时，则会默认注册bind、update钩子
    if (type === 'directive' && typeof definition === 'function') {
        definition = { bind: definition, update: definition }
    }
    return definition
}
```

如果以下指令：

```javaScript
Vue.directive('demo', {
    bind: function (el, binding, vnode) {
        el.focus()
    }
})
```

在`Vue`对进行模板解析生成`AST`时，就会对指令进行处理——统一处理attr——`parse-parseHTML-parseStartTag`。

```javaScript
"with(this){return _c('div',{attrs:{"id":"app"}},[_c('input',{directives:[{name:"demo",rawName:"v-demo"}],attrs:{"type":"text"}})])}"
```

然后在`_patch_`中`createEle`时:

```javaScript
//继续执行子节点
createChildren(vnode, children, insertedVnodeQueue)
// 如果存在指令,则创建钩子
// data = {directive: [{name:"demo", rawName: "v-demo"}, attrs:{type:"text"}]}
if (isDef(data)) {
    invokeCreateHooks(vnode, insertedVnodeQueue)
}
// 插入父节点
insert(parentElm, vnode.elm, refElm)
```
`invokeCreateHooks`则是：

```javaScript
const cbs = {}
// hooks是[create", "activate", "update", "remove", "destroy"]
// modules是一个数组 每个都有一个create 方法分别对应style/class/ref等属性
// export default {
//  create: updateDirectives,
//  update: updateDirectives,
//  destroy: function unbindDirectives (vnode: VNodeWithData) {
//    updateDirectives(vnode, emptyNode)
//  }
// }
const { modules, nodeOps } = backend
for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = []
    for (j = 0; j < modules.length; ++j) {
        if (isDef(modules[j][hooks[i]])) {
        cbs[hooks[i]].push(modules[j][hooks[i]])
        }
    }
}
function invokeCreateHooks (vnode, insertedVnodeQueue) {
    // 这个就是之前所有的create钩子执行
    // 具体到directive的create钩子指的就是updateDirectives
    for (var i$1 = 0; i$1 < cbs.create.length; ++i$1) {
        // 需要注意的是，这个create
        cbs.create[i$1](emptyNode, vnode);
    }
    i = vnode.data.hook; // Reuse variable
    if (isDef(i)) {
        if (isDef(i.create)) { i.create(emptyNode, vnode); }
        if (isDef(i.insert)) { insertedVnodeQueue.push(vnode); }
    }
}
```

`updateDirectives`是：

```javaScript
function updateDirectives (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (oldVnode.data.directives || vnode.data.directives) {
    _update(oldVnode, vnode)
  }
}

function _update (oldVnode, vnode) {
  const isCreate = oldVnode === emptyNode
  const isDestroy = vnode === emptyNode
  // 获取指令
  // 比如：v-demo:{name:'demo', ref:{bind:() => {}}}
  const oldDirs = normalizeDirectives(oldVnode.data.directives, oldVnode.context)
  const newDirs = normalizeDirectives(vnode.data.directives, vnode.context)

  const dirsWithInsert = []
  const dirsWithPostpatch = []

  let key, oldDir, dir
  for (key in newDirs) {
    oldDir = oldDirs[key]
    dir = newDirs[key]
    if (!oldDir) {
      // 第一次创建指令
      callHook(dir, 'bind', vnode, oldVnode)
      if (dir.def && dir.def.inserted) {
        // 指令存数组
        dirsWithInsert.push(dir)
      }
    } else {
      // 已有指令，所以是update
      dir.oldValue = oldDir.value
      dir.oldArg = oldDir.arg
      callHook(dir, 'update', vnode, oldVnode)
      if (dir.def && dir.def.componentUpdated) {
        dirsWithPostpatch.push(dir)
      }
    }
  }

  if (dirsWithInsert.length) {
    // 指令的inserted钩子封进函数callInsert
    // 也说明是所有钩子都是节点触发的，而不是一个个执行的
    const callInsert = () => {
      for (let i = 0; i < dirsWithInsert.length; i++) {
        callHook(dirsWithInsert[i], 'inserted', vnode, oldVnode)
      }
    }
    if (isCreate) {
      // 合并钩子到当前vue component
      mergeVNodeHook(vnode, 'insert', callInsert)
    } else {
      callInsert()
    }
  }

  if (dirsWithPostpatch.length) {
    mergeVNodeHook(vnode, 'postpatch', () => {
      for (let i = 0; i < dirsWithPostpatch.length; i++) {
        callHook(dirsWithPostpatch[i], 'componentUpdated', vnode, oldVnode)
      }
    })
  }

  if (!isCreate) {
    for (key in oldDirs) {
      if (!newDirs[key]) {
        // no longer present, unbind
        callHook(oldDirs[key], 'unbind', oldVnode, oldVnode, isDestroy)
      }
    }
  }
}
```

把所有指令的钩子都装进队列`insertedVnodeQueue`，在最后通过`invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)`触发。

> 注意：更新指令时，钩子是及时触发的。

参考：[【Vue原理】Directives - 源码版](https://zhuanlan.zhihu.com/p/57089620), [vue自定义指令--directive](https://segmentfault.com/a/1190000018767046)

## v-model

本质上，`v-model`是一个语法糖，它相当于输入框会绑定两个属性`v-on`与`:value`。


