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

