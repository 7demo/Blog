# initMixin

```javascript
// 传入Vue的类，然后在Vue的prototype挂在_init方法。
export function initMixin (Vue: Class<Component>) {
  // 初始化 传入options的对象
  Vue.prototype._init = function (options?: Object) {
    // 当前vue的实例赋值到vm上
    const vm: Component = this
    // a uid
    // 当前Vue生成一个实例，则_uid增加1
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    // 非生产模式下，并且开启了记录组件性能
    // mark方法是使用perf方法
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    // 一个标识，避免处于被观察？？？
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // 优化组件内部实例
      // 由于动态配置慢，切内部组件不需要优化
      // 初始化内部组件
      initInternalComponent(vm, options)
    } else {
      // 合并vm实例的构造函数的配置项
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

初始化内部组件

```javascript
// 初始化内部组件，传入两个参数 一个Vue的实例，一个配置项
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  // 获取vue的实例的配置项目
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  // 父节点，（为了更快速的操作???）
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  // 节点组件的配置项
  const vnodeComponentOptions = parentVnode.componentOptions
  // 属性data
  opts.propsData = vnodeComponentOptions.propsData
  // 父节点监听函数
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  // 组件标识
  opts._componentTag = vnodeComponentOptions.tag

  // render函数
  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}
```

initProxy给vue实例挂在_renderProxy。在支持`proxy`模式下，为一个新的`Proxy`实例，否则为自身实例。

```javascript
initProxy = function initProxy (vm) {
  // 支持proxy语法的话
  if (hasProxy) {
    // determine which proxy handler to use
    const options = vm.$options
    const handlers = options.render && options.render._withStripped
      ? getHandler
      : hasHandler
    vm._renderProxy = new Proxy(vm, handlers)
  } else {
    vm._renderProxy = vm
  }
}
```
