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

[配置文件](/201906/vue-config.md)


#### 启动文件

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

初始化方法在`core/instance/init`中的`initMixin`方法。主要作用：

+ 在`Vue`的原型链挂载[`_init`](/201906/vue-initMixin.md)方法。

+ 初始化`lifecycle`[生命周期事件](/201906/vue-lifecycle.md)。

+ 初始化`initEvents`事件，[在`vue`创建`events`对象](/201906/vue-events.md)

+ 初始化`initRender`, [`vue`实例挂载`createElement`](/201906/vue-render.md)

+ 初始化`initInjections`与`initProvide`，分别在`vue`实例上挂载[`inject`与`__provided`](/201906/vue-inject.md)

+ 初始化状态`initState`，初始化[`prop`/`data`/`methods`/`watch`](/201906/vue-state.md)。