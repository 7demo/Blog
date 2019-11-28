# Vue之插件

## 用法：

1， 插件要暴露一个`install`静态方法。

2，在新建`vue`实例前，`vue`通过`use`方法。

```javaScript
plugin.install = function(vue, options) {

}
vue.use(plugin)
```

## 源码

在`initGlobalAPI`时，会调用`initUse`。

```javascript
Vue.use = function (plugin: Function | Object) {
    // 插件数组，避免重复注册
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    // 把Vue放进第一次参数中
    args.unshift(this)
    // install方法存在的话，则直接调用install
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    // 如果没有install也是可以的。
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
}
```
