# Vue源码学习

> 先通过`git`克隆vue的源码到本地，`vue`的版本号为`2.6.10`。

## 目录结构

├── LICENSE
├── benchmarks
├── dist # rollup编译后产生的文件
├── examples
├── flow
├── package.json
├── packages
├── scripts
│   ├── alias.js
│   ├── build.js
│   ├── config.js
│   ├── feature-flags.js
│   ├── gen-release-note.js
│   ├── get-weex-version.js
│   ├── git-hooks
│   ├── release-weex.sh
│   ├── release.sh
│   └── verify-commit-msg.js
├── src
│   ├── compiler
│   ├── core
│   ├── platforms
│   ├── server
│   ├── sfc
│   └── shared
├── test
├── types
└── yarn.lock

根据`package.json`中的`rollup -w -c scripts/config.js --environment TARGET:web-full-dev`在`scripts/config.js`中的相关配置为：

```
'web-full-dev': {
	entry: resolve('web/entry-runtime-with-compiler.js'), # 入口文件，resolve函数查alias.js来确定web为src/platforms/web
	dest: resolve('dist/vue.js') # 编译产生的文件
},
```

入口文件即为：`/src/platforms/web/entry-runtime-with-compiler.js`。

#### `/src/core/config.js`

`Vue`相关配置信息

```javascript
optionMergeStrategies: 合并策略

silent: 是否关闭控制台警告

productionTip: 是否开启生产模式下的引导提示

devtools： 是否开启devtools

performance： 是否记录组件性能

ignoredElements：忽略自定义元素

keyCodes： 为v-on自定义别名

isReservedTag：检查标签是否被保留的，不能被使用为组件。

isReservedAttr： 检查属性是被保留。

isUnknownElement： 检测元素是未知的。

getTagNamespace: 获得标签的命名空间。

parsePlatformTagName： 解析平台标签名字

mustUseProp：检测是否属性必须用prototype绑定

async：开启同步更新，主要是为了vue test。

```

#### 起始

`Vue`方法是最开始是在`core/instance/index`中声明：

```javascript
// 环境变量如果不是production并且不是Vue的实例，则报错。否则则初始化
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

初始化方法在`core/instance/init`中的`initMixin`方法：

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
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.

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
