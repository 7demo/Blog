# Vue之component

> 在Vue中，有三种注册组件方式：Vue.extend, Vue.component及通过路由挂载。

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

我们还是需要从`patch`方法开始，它是用来生成`Vnode`的。在方法`patch`中`createElm`中：

```javaScript
function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {

    vnode.isRootInsert = !nested // for transition enter check
    // 如果createComponetent返回true，表示当前元素是一个组件
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }
    // 进而创建子元素
    createChildren(vnode, children, insertedVnodeQueue)
  }
// 创建子元素
function createChildren (vnode, children, insertedVnodeQueue) {
    if (Array.isArray(children)) {
        if (process.env.NODE_ENV !== 'production') {
        checkDuplicateKeys(children)
        }
        // 遍历子元素包括，包括组件元素，又掉起createElm
        for (let i = 0; i < children.length; ++i) {
        createElm(children[i], insertedVnodeQueue, vnode.elm, null, true, children, i)
        }
    } else if (isPrimitive(vnode.text)) {
        nodeOps.appendChild(vnode.elm, nodeOps.createTextNode(String(vnode.text)))
    }
}
```

当时组件元素时，

```javascript
function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
    let i = vnode.data
    if (isDef(i)) {
      const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
      if (isDef(i = i.hook) && isDef(i = i.init)) {
        i(vnode, false /* hydrating */)
      }
      // after calling the init hook, if the vnode is a child component
      // it should've created a child instance and mounted it. the child
      // component also has set the placeholder vnode's elm.
      // in that case we can just return the element and be done.
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
```


