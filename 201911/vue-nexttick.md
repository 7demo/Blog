# Vue之nexttick与异步渲染

`Vue`中的`nickTick`代码在`src/core/util/next-tick.js`中。精简下，把主要代码贴出来。

```javascript

// 存放回调
// 其实多通过暴露出去的nextTick方法，把待执行的订阅Watcher存放到这。当然也包括直接通过this.$nextTick的回调
const callbacks = []
// 是否在执行回调
// 在每个人物阶段，如果已经在执行flushCallbacks，就是该批次回调时，则跳过。
let pending = false

// 清空callbacks，也就是执行
function flushCallbacks () {
  pending = false
  // 此处是复制当前队列中的回调去执行，免得在执行时依然被插入回调
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
let timerFunc
// 如果当前环境支持Promise，则直接在Promise.reslove中执行
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    // 在ios UIWebViews中，会一直处于微任务等待中，所以需要执行一个空函数来跳出
    if (isIOS) setTimeout(noop)
  }
// 如果支持MutationObserver
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  // 去监测一个文本节点来触发进入微任务去执行flushCallbacks
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
// 使用setImmediate
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
// 使用setTimeout
} else {
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
// 最终暴露出去的方法
// 主要是把回调函数推入callbacks
// 然后看是否在执行，如果没有执行清空队列清空则直接执行
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  //
  callbacks.push(() => {
    if (cb) {
      cb.call(ctx)
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

```

在`timerFunc`的实现上，先后有`Promise/MutationObserver/Immediate/setTimeout`四种方法，依次进行降级。

`Promise`就是直接调用`reslove`来把`flushCallbacks`放入微任务中。`MutationObserver`则是通过观察文本节点的变化来把`flushCallbacks`放入微任务中执行，至于优先级不如`Promise`高的原因则是因为`IOS`兼容性问题。

## Vue异步渲染

通过`nextTick`的源码可以知道只要调用`nextTick`就会去把`flushCallbacks`放入微任务中，那么如何保证同一个`marcroTask`多次更改数据只渲染一次呢。我们可以在订阅器`Watcher`中一窥端倪。

```javascript
update () {
    /* istanbul ignore else */
    if (this.lazy) {
        this.dirty = true
    } else if (this.sync) {
        this.run()
    } else {
        queueWatcher(this)
    }
}
```

执行时，调用了`queueWatcher`方法。

```javascript
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  // 监测watcher是否已存在
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      // 开始调用nextTick
      nextTick(flushSchedulerQueue)
    }
  }
}
```

其实在同一task中一旦调用`queueWatcher`，就触发了`nextTick`。只不过触发后依然可以把当前task中的`watcher`加入到`queue`中，但是由于`waiting`值得问题，就不会再调用`nextTick`了。这就实现了多个同步操作，只会触发一次`nextTick`。

最后，在浏览器中，每个`macroTask`后必然会清空`microTask`(vue中有一个最大值为100，超过则延后执行)，然后就会`render ui`。这就保证了性能、渲染的及时性。

如果在不支持`Promsie`/`MuatationObserver`，则会用到`setTimeout`，就会等到下次个`macroTask`执行。

参考：

[深度解析vue的$nextTick的实现原理](https://github.com/FlyDreame/2m-blog/issues/2)

[Vue源码详解之nextTick：MutationObserver只是浮云，microtask才是核心](https://github.com/Ma63d/vue-analysis/issues/6)
