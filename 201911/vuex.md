# vuex

> 以下是基于`vuex` 3.1.2版本进行分析。整个包从github拉下来后，可以看到入口文件为`src/index.js`.

```javaScript
export default {
  Store,
  install,
  version: '__VERSION__',
  mapState,
  mapMutations,
  mapGetters,
  mapActions,
  createNamespacedHelpers
}
```

整个`vuex`暴露出来一个对象，包含`install/Store/...`。特别是`install`，是挂载到`vue`必备方法。

```javaScript
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue)
}
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])
  // 根据版本号选择对应的方式把vuexInit挂载到beforeCreate钩子上。
  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit
      _init.call(this, options)
    }
  }

  /**
   * Vuex init hook, injected into each instances init hooks list.
   */
  // 此处的this 为vue的实例对象
  function vuexInit () {
    const options = this.$options
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```

`install`就是在`vue`的`beforeCreate`钩子上执行`vueInit`。主要是为从`options`上获取`store`挂到`vue.$store`上。

在使用上，我们都是如下。看到`options.store`是一个`Store`的实例。

```javaScript
// store.js
Vue.use(Vuex)
export default new Vuex.Store({
  state,
  getters,
  actions,
  mutations
})

import store from './store'
new Vue({
  el: '#app',
  store,
  render: h => h(Counter)
})
```

要了解整个`Store`，需要先了解下`module`——`module/module.js、module/module-collection.js`。这个`module`就是我们使用中`vuex`的模块，可能如下：

```javaScript
export default new Vuex.Store({
  modules: {
    cart,
    products
  },
  strict: debug,
  plugins: debug ? [createLogger()] : []
})
```

在`vuex`创建实例的时候会注册`module`——`this._modules = new ModuleCollection(options)`。

```javaScript
export default class ModuleCollection {
  constructor (rawRootModule) {
    // register root module (Vuex.Store options)
    this.register([], rawRootModule, false)
  }
  register (path, rawModule, runtime = true) {
    if (process.env.NODE_ENV !== 'production') {
      assertRawModule(path, rawModule)
    }
    const newModule = new Module(rawModule, runtime)
    if (path.length === 0) {
      this.root = newModule
    } else {
      const parent = this.get(path.slice(0, -1))
      parent.addChild(path[path.length - 1], newModule)
    }

    // 注册模块
    if (rawModule.modules) {
      forEachValue(rawModule.modules, (rawChildModule, key) => {
        this.register(path.concat(key), rawChildModule, runtime)
      })
    }
  }
}
```

这注册模块的时候，做了判断，如果是顶层模块，即是没有模块名字，也会注册为`根模块`，其他模块则会循环挂在。接着看`Store`初始化的时候。

```javaScript
const { dispatch, commit } = this
this.dispatch = function boundDispatch (type, payload) {
    return dispatch.call(store, type, payload)
}
this.commit = function boundCommit (type, payload, options) {
    return commit.call(store, type, payload, options)
}
```

是把`Store`的`dispatch`/`commit`注册为静态法方法。

```javaScript
// 三个参数分别对应key、payload
commit (_type, _payload, _options) {
    // check object-style commit
    // 校验参数，返回 type, payload, options
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    // 根据type拿到 预先定义的mutationdhander
    const entry = this._mutations[type]
    if (!entry) {
      if (process.env.NODE_ENV !== 'production') {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    // 执行mutationdhander
    this._withCommit(() => {
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
    // 开发工具相关订阅
    this._subscribers.forEach(sub => sub(mutation, this.state))
}


dispatch (_type, _payload) {
    // check object-style dispatch
    const {
      type,
      payload
    } = unifyObjectStyle(_type, _payload)

    const action = { type, payload }
    const entry = this._actions[type]

    // before钩子
    try {
      this._actionSubscribers
        .filter(sub => sub.before)
        .forEach(sub => sub.before(action, this.state))
    } catch (e) {
      if (process.env.NODE_ENV !== 'production') {
        console.warn(`[vuex] error in before action subscribers: `)
        console.error(e)
      }
    }
    // Promise.js
    const result = entry.length > 1
      ? Promise.all(entry.map(handler => handler(payload)))
      : entry[0](payload)
    // 这个地方action也是支持异步的原因
    return result.then(res => {
      try {
        this._actionSubscribers
          .filter(sub => sub.after)
          .forEach(sub => sub.after(action, this.state))
      } catch (e) {
        if (process.env.NODE_ENV !== 'production') {
          console.warn(`[vuex] error in after action subscribers: `)
          console.error(e)
        }
      }
      return res
    })
}
```

接下来会初始化`state`, 它是从根模块的`state`开始，子模块的`state`会递归执行`installModule(this, state, [], this._modules.root)`:

```javaScript
function installModule (store, rootState, path, module, hot) {
  const isRoot = !path.length
  const namespace = store._modules.getNamespace(path)

  // register in namespace map
  if (module.namespaced) {
    if (store._modulesNamespaceMap[namespace] && process.env.NODE_ENV !== 'production') {
      console.error(`[vuex] duplicate namespace ${namespace} for the namespaced module ${path.join('/')}`)
    }
    store._modulesNamespaceMap[namespace] = module
  }

  // 如果是子模块，调用vue.set方法来设置state
  if (!isRoot && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    store._withCommit(() => {
      Vue.set(parentState, moduleName, module.state)
    })
  }

  const local = module.context = makeLocalContext(store, namespace, path)
  // 依次挂在mutation/getter/action
  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })

  module.forEachAction((action, key) => {
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })

  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })
  // 递归挂在子模块
  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

然后是`resetStoreVM`, 主要是为了可以响应式使用state中的值，比如`mapGetter`。

```javaScript
function resetStoreVM (store, state, hot) {
  const oldVm = store._vm

  // bind store public getters
  store.getters = {}
  // reset local getters cache
  store._makeLocalGettersCache = Object.create(null)
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  // 遍历getter中的值，通过defineProperty，获取$store.gettters获取值会走到store.vm，并且会存放到computed对象中，之后就是computed属性
  // 所以是计算属性生效
  forEachValue(wrappedGetters, (fn, key) => {
    computed[key] = partial(fn, store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true // for local getters
    })
  })
  // 新建一个vue实例，把state绑定到 上。
  const silent = Vue.config.silent
  Vue.config.silent = true
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  Vue.config.silent = silent

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store)
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(() => {
        oldVm._data.$$state = null
      })
    }
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```

最后则是挂在插件

```javaScript
plugins.forEach(plugin => plugin(this))
```


然后再看看三大对象：`mapMutations, mapGetters, mapActions`.

```javaScript

export const mapMutations = normalizeNamespace((namespace, mutations) => {
  // normalizeMap([1, 2, 3]) => [ { key: 1, val: 1 }, { key: 2, val: 2 }, { key: 3, val: 3 } ]
  // 遍历所有mutation，
  normalizeMap(mutations).forEach(({ key, val }) => {
    res[key] = function mappedMutation (...args) {
      // Get the commit method from store
      // this指当前vue
      let commit = this.$store.commit
      if (namespace) {
        const module = getModuleByNamespace(this.$store, 'mapMutations', namespace)
        if (!module) {
          return
        }
        commit = module.context.commit
      }
      return typeof val === 'function'
        ? val.apply(this, [commit].concat(args))
        : commit.apply(this.$store, [val].concat(args))
    }
  })
  // 最后返回一个 {key: mappedMutation(...args){}}
  // 所以我们最后使用该方法时，是执行的mappedMutation(...args){}
  return res
})
```
