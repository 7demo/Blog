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


#### `/src/platforms/web/entry-runtime-with-compiler.js`

```javascript
/**
 * config 是相关配置信息
 */
import config from 'core/config'
import { warn, cached } from 'core/util/index'
import { mark, measure } from 'core/util/perf'

import Vue from './runtime/index'
import { query } from './util/index'
import { compileToFunctions } from './compiler/index'
import { shouldDecodeNewlines, shouldDecodeNewlinesForHref } from './util/compat'

const idToTemplate = cached(id => {
  const el = query(id)
  return el && el.innerHTML
})

const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}

/**
 * Get outerHTML of elements, taking care
 * of SVG elements in IE as well.
 */
function getOuterHTML (el: Element): string {
  if (el.outerHTML) {
    return el.outerHTML
  } else {
    const container = document.createElement('div')
    container.appendChild(el.cloneNode(true))
    return container.innerHTML
  }
}

Vue.compile = compileToFunctions

export default Vue
```

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
