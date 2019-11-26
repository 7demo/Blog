# Vue之component

> 在Vue中，可通过：Vue.extend, Vue.component创建组件。

### Vue.extend

常用用法：

```JavaScript
// 创建组件
var MyComponent = Vue.extend({
    name: 'my-component',
    template: '<div>A custom component!</div>'
});
// 注册
Vue.component('my-component', MyComponent);
```

关于`Vue.extend`的实现在`src/core/global-api/extend.js`中：

```JavaScript
Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
    const Super = this
    const SuperId = Super.cid
    const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
    // 如果已有的话
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }

    const name = extendOptions.name || Super.options.name

    // Sub最后将导出一个方法
    const Sub = function VueComponent (options) {
      this._init(options)
    }
    // 复制父组件，也就是Vue的原型链
    Sub.prototype = Object.create(Super.prototype)
    Sub.prototype.constructor = Sub
    Sub.cid = cid++
    // 把vue的option与当前组件进行合并，这个也是组件内可以使用root组件方法的原因。
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    )
    // 指向父组件
    Sub['super'] = Super
    if (Sub.options.props) {
      initProps(Sub)
    }
    if (Sub.options.computed) {
      initComputed(Sub)
    }

    // allow further extension/mixin/plugin usage
    // 把Vue的extend、mixin、use方法继承，方便在内部使用组件
    Sub.extend = Super.extend
    Sub.mixin = Super.mixin
    Sub.use = Super.use

    // create asset registers, so extended classes
    // can have their private assets too.
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type]
    })
    // enable recursive self-lookup
    if (name) {
      Sub.options.components[name] = Sub
    }

    // keep a reference to the super options at extension time.
    // later at instantiation we can check if Super's options have
    // been updated.
    Sub.superOptions = Super.options
    Sub.extendOptions = extendOptions
    Sub.sealedOptions = extend({}, Sub.options)

    // cache constructor
    cachedCtors[SuperId] = Sub
    return Sub
}
```

`extend`方法就是返回一个`Sub`方法，继承了`Vue`的原型，包括`_init`方法。接下来就是注册。

在`src/core/global-api/index.js`初始化全局方法`initGlobalAPI`时，有调用`initAssetRegisters`。


```javascript
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
export function initAssetRegisters (Vue: GlobalAPI) {
  // 分别注册Vue.component、Vue.directive、Vue.filter
  ASSET_TYPES.forEach(type => {
    Vue[type] = function (
      id: string,
      definition: Function | Object
    ): Function | Object | void {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id)
        }
        // 如果component是通过vue.component(name, obj)形式上调用的话，则把definition交由Vue.extend声明一次
        // 当然如果直接是一个组件(VueComponent function)的话 则不需要处理
        // 这里也直接明白了：`Vue.component('name', {})`的形式其本质
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id
          definition = this.options._base.extend(definition)
        }
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition }
        }
        this.options[type + 's'][id] = definition
        return definition
      }
    }
  })
}
```

以上，我们把组件创建好，会在`vue.options`存在一个以组件名称命名的组件。下面将是渲染。

我们还是需要从`$mount`方法开始：

```javaScript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

+ 1，开始在`$mount`方法中调用`mountComponent`

```javaScript
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
new Watcher(vm, updateComponent, noop, {
  before () {
    if (vm._isMounted && !vm._isDestroyed) {
      callHook(vm, 'beforeUpdate')
    }
  }
}, true /* isRenderWatcher */)
```
+ 2，在`mountComponent`中拥有`updateComponent`方法，此方法创建一个订阅器，响应数据的变化。在内部有两个关键的方法`_render`与`_update`。

```javaScript
Vue.prototype._render = function (): VNode {
    const vm: Component = this
    const { render, _parentVnode } = vm.$options
    vnode = render.call(vm._renderProxy, vm.$createElement)
}
```

+ 3, `vm.$createElement`其实就是`_createElement`方法。

```javaScript
vnode = createComponent(Ctor, data, context, children, tag)
```

+ 4，在遍历创建vdom的时候，遇到组件，调用`createComponent`，注意在`vnode.componentOptions.Ctor`赋值组件。

```javascript
export function createComponent (
  Ctor: Class<Component> | Function | Object | void,
  data: ?VNodeData,
  context: Component,
  children: ?Array<VNode>,
  tag?: string
): VNode | Array<VNode> | void {
  if (isUndef(Ctor)) {
    return
  }

  const baseCtor = context.$options._base

  // plain options object: turn it into a constructor
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor)
  }

  // if at this stage it's not a constructor or an async component factory,
  // reject.
  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Invalid Component definition: ${String(Ctor)}`, context)
    }
    return
  }

  // 异步组件
  let asyncFactory
  if (isUndef(Ctor.cid)) {
    asyncFactory = Ctor
    Ctor = resolveAsyncComponent(asyncFactory, baseCtor)
    if (Ctor === undefined) {
      // return a placeholder node for async component, which is rendered
      // as a comment node but preserves all the raw information for the node.
      // the information will be used for async server-rendering and hydration.
      return createAsyncPlaceholder(
        asyncFactory,
        data,
        context,
        children,
        tag
      )
    }
  }

  data = data || {}

  resolveConstructorOptions(Ctor)

  if (isDef(data.model)) {
    transformModel(Ctor.options, data)
  }

  // extract props
  const propsData = extractPropsFromVNodeData(data, Ctor, tag)

  // functional component
  if (isTrue(Ctor.options.functional)) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }

  const listeners = data.on
  data.on = data.nativeOn

  if (isTrue(Ctor.options.abstract)) {
    // abstract components do not keep anything
    // other than props & listeners & slot

    // work around flow
    const slot = data.slot
    data = {}
    if (slot) {
      data.slot = slot
    }
  }

  // 挂上componentVNodeHooks的init、prepatch、insert、destory钩子
  // 在__patch__中createComponent中使用
  installComponentHooks(data)
  // return a placeholder vnode
  const name = Ctor.options.name || tag
  // 创建tag为vue-component-x-name格式的组件节点vdom
  const vnode = new VNode(
    `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
    data, undefined, undefined, undefined, context,
    { Ctor, propsData, listeners, tag, children },
    asyncFactory
  )
  return vnode
}
```

+ 5，以上`_render`创建了Vdom，包括组件。

```javaScript
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    vm.$el = vm.__patch__(prevVnode, vnode)
  }
```

+ 6, `_update`中调用`__patch__`。

```javascript
createPatchFunction() {
  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    createElm(
      vnode,
      insertedVnodeQueue,
      // extremely rare edge case: do not insert if old element is in a
      // leaving transition. Only happens when combining transition +
      // keep-alive + HOCs. (#4590)
      oldElm._leaveCb ? null : parentElm,
      nodeOps.nextSibling(oldElm)
    )

    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch)
    return vnode.elm
  }
}

function createElm (
  vnode,
  insertedVnodeQueue,
  parentElm,
  refElm,
  nested,
  ownerArray,
  index
) {
  if (isDef(vnode.elm) && isDef(ownerArray)) {
    // This vnode was used in a previous render!
    // now it's used as a new node, overwriting its elm would cause
    // potential patch errors down the road when it's used as an insertion
    // reference node. Instead, we clone the node on-demand before creating
    // associated DOM element for it.
    vnode = ownerArray[index] = cloneVNode(vnode)
  }

  vnode.isRootInsert = !nested // for transition enter check
  // 如果是组件
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    return
  }

  const data = vnode.data
  const children = vnode.children
  const tag = vnode.tag
  if (isDef(tag)) {
    if (process.env.NODE_ENV !== 'production') {
      if (data && data.pre) {
        creatingElmInVPre++
      }
      if (isUnknownElement(vnode, creatingElmInVPre)) {
        warn(
          'Unknown custom element: <' + tag + '> - did you ' +
          'register the component correctly? For recursive components, ' +
          'make sure to provide the "name" option.',
          vnode.context
        )
      }
    }

    vnode.elm = vnode.ns
      ? nodeOps.createElementNS(vnode.ns, tag)
      : nodeOps.createElement(tag, vnode)
    setScope(vnode)

    /* istanbul ignore if */
    if (__WEEX__) {
      // in Weex, the default insertion order is parent-first.
      // List items can be optimized to use children-first insertion
      // with append="tree".
      const appendAsTree = isDef(data) && isTrue(data.appendAsTree)
      if (!appendAsTree) {
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        insert(parentElm, vnode.elm, refElm)
      }
      createChildren(vnode, children, insertedVnodeQueue)
      if (appendAsTree) {
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue)
        }
        insert(parentElm, vnode.elm, refElm)
      }
    } else {
      createChildren(vnode, children, insertedVnodeQueue)
      if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue)
      }
      insert(parentElm, vnode.elm, refElm)
    }

    if (process.env.NODE_ENV !== 'production' && data && data.pre) {
      creatingElmInVPre--
    }
  } else if (isTrue(vnode.isComment)) {
    vnode.elm = nodeOps.createComment(vnode.text)
    insert(parentElm, vnode.elm, refElm)
  } else {
    vnode.elm = nodeOps.createTextNode(vnode.text)
    insert(parentElm, vnode.elm, refElm)
  }
}

function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
    // 注意此处一步步把i=data.hook.init 赋值给i
    // 其实是在执行 init(vnode, false)
    if (isDef(i = i.hook) && isDef(i = i.init)) {
      i(vnode, false /* hydrating */)
    }
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue)
      insert(parentElm, vnode.elm, refElm)
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm)
      }
      return true
    }
  }
}

const componentVNodeHooks = {
  init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      // 在此会创建组件的实例
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      )
      console.log(111111, vnode.elm, vnode)
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  }
}

export function createComponentInstanceForVnode (
  vnode: any, // we know it's MountedComponentVNode but flow doesn't
  parent: any, // activeInstance in lifecycle state
): Component {
  const options: InternalComponentOptions = {
    _isComponent: true,
    _parentVnode: vnode,
    parent
  }
  // check inline-template render functions
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  // 终于在这儿，见到了新建实例组件，进而调用_init。
  // Ctor是在_render, _createElement->createComponent中赋值
  return new vnode.componentOptions.Ctor(options)
}
```

+ 7, `_patch_`中，创建vnode过程中，如果是组件的话执行`createComponent`，进而初始化。


## 异步组件

在`_render`执行`createComponent`时，有一个关于异步组件的处理：

```javaScript
let asyncFactory
// 如果没有cid 走异步流程
if (isUndef(Ctor.cid)) {
  asyncFactory = Ctor
  Ctor = resolveAsyncComponent(asyncFactory, baseCtor)
  if (Ctor === undefined) {
    // return a placeholder node for async component, which is rendered
    // as a comment node but preserves all the raw information for the node.
    // the information will be used for async server-rendering and hydration.
    // 创建一个占位符，注释存在。
    return createAsyncPlaceholder(
      asyncFactory,
      data,
      context,
      children,
      tag
    )
  }
}

export function resolveAsyncComponent (
  factory: Function,
  baseCtor: Class<Component>
): Class<Component> | void {
  if (isTrue(factory.error) && isDef(factory.errorComp)) {
    return factory.errorComp
  }

  if (isDef(factory.resolved)) {
    return factory.resolved
  }
  // 如果当前组件实例存在
  const owner = currentRenderingInstance
  if (owner && isDef(factory.owners) && factory.owners.indexOf(owner) === -1) {
    // already pending
    factory.owners.push(owner)
  }

  if (isTrue(factory.loading) && isDef(factory.loadingComp)) {
    return factory.loadingComp
  }

  if (owner && !isDef(factory.owners)) {
    const owners = factory.owners = [owner]
    let sync = true
    let timerLoading = null
    let timerTimeout = null

    ;(owner: any).$on('hook:destroyed', () => remove(owners, owner))

    const forceRender = (renderCompleted: boolean) => {
      for (let i = 0, l = owners.length; i < l; i++) {
        (owners[i]: any).$forceUpdate()
      }

      if (renderCompleted) {
        owners.length = 0
        if (timerLoading !== null) {
          clearTimeout(timerLoading)
          timerLoading = null
        }
        if (timerTimeout !== null) {
          clearTimeout(timerTimeout)
          timerTimeout = null
        }
      }
    }
    // once表示只执行一次，正常加载时
    const resolve = once((res: Object | Class<Component>) => {
      // cache resolved
      factory.resolved = ensureCtor(res, baseCtor)
      // invoke callbacks only if this is not a synchronous resolve
      // (async resolves are shimmed as synchronous during SSR)
      if (!sync) {
        forceRender(true)
      } else {
        owners.length = 0
      }
    })

    // 未能正常加载
    const reject = once(reason => {
      process.env.NODE_ENV !== 'production' && warn(
        `Failed to resolve async component: ${String(factory)}` +
        (reason ? `\nReason: ${reason}` : '')
      )
      if (isDef(factory.errorComp)) {
        factory.error = true
        forceRender(true)
      }
    })
    // 执行factory函数
    // 其实这个函数是经过webpack处理的
    const res = factory(resolve, reject)

    if (isObject(res)) {
      // 如果Promise
      if (isPromise(res)) {
        // () => Promise
        if (isUndef(factory.resolved)) {
          res.then(resolve, reject)
        }
      } else if (isPromise(res.component)) {
        res.component.then(resolve, reject)

        if (isDef(res.error)) {
          factory.errorComp = ensureCtor(res.error, baseCtor)
        }

        if (isDef(res.loading)) {
          factory.loadingComp = ensureCtor(res.loading, baseCtor)
          if (res.delay === 0) {
            factory.loading = true
          } else {
            timerLoading = setTimeout(() => {
              timerLoading = null
              if (isUndef(factory.resolved) && isUndef(factory.error)) {
                factory.loading = true
                forceRender(false)
              }
            }, res.delay || 200)
          }
        }

        if (isDef(res.timeout)) {
          timerTimeout = setTimeout(() => {
            timerTimeout = null
            if (isUndef(factory.resolved)) {
              reject(
                process.env.NODE_ENV !== 'production'
                  ? `timeout (${res.timeout}ms)`
                  : null
              )
            }
          }, res.timeout)
        }
      }
    }

    sync = false
    // return in case resolved synchronously
    return factory.loading
      ? factory.loadingComp
      : factory.resolved
  }
}

```

异步组件可以如下使用：

```javaScript
const AsyncComponent = () => ({
  // 加载组件（最终应该返回一个 Promise）
  component: import('./MyComponent.vue'),
  // 异步组件加载中(loading)，展示为此组件
  loading: LoadingComponent,
  // 加载失败，展示为此组件
  error: ErrorComponent,
  // 展示 loading 组件之前的延迟时间。默认：200ms。
  delay: 200,
  // 如果提供 timeout，并且加载用时超过此 timeout，
  // 则展示错误组件。默认：Infinity。
  timeout: 3000
})
```

### Keep-alive

```javaScript
init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      )
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  }

insert (vnode: MountedComponentVNode) {
    const { context, componentInstance } = vnode
    if (!componentInstance._isMounted) {
      componentInstance._isMounted = true
      callHook(componentInstance, 'mounted')
    }
    if (vnode.data.keepAlive) {
      if (context._isMounted) {
        // vue-router#1212
        // During updates, a kept-alive component's child components may
        // change, so directly walking the tree here may call activated hooks
        // on incorrect children. Instead we push them into a queue which will
        // be processed after the whole patch process ended.
        queueActivatedComponent(componentInstance)
      } else {
        activateChildComponent(componentInstance, true /* direct */)
      }
    }
  }
```

在初始化组件时，如果`vnode.data.keepAlive`为`true`切组件存在时，则直接执行`prepatch`方法。并且最后`insert`时做兼容处理，主要时根据`iskeepalive`来确定是否激活钩子。它的生命周期：`created-> mounted-> activated -> deactivated -> activated...`
