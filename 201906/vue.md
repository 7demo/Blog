# Vue源码学习

> 先通过`git`克隆vue的源码到本地，`vue`的版本号为`2.6.10`。

## 目录结构

.
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