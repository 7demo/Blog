# koa-compose

> koa-compose 是把一个包含多个中间件的数组，转成一个中间件。用来实现洋葱模型。

源码如下：

```javascript
function compose (middleware) {
  // 判断传入的中间件是否是一个数组
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  // 判断数组中的每个中间件是一个函数
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }
  // 返回一个函数，毕竟最终是把多个中间件函数转成一个函数，所以也需要返回一个函数。
  // 两个参数一个是上下文，一个是next函数。 
  return function (context, next) {
    // last called middleware #
    let index = -1
    // 从第一个中间件第一个函数开始执行。
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      // 如果i等于数组的长度，表示数组已经执行一遍, 不取中间件直接执行next
      // 如果fn不存在直接结束。
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        // 返回fn的执行结果。
        // 其中，dispatch.bind(null, i + 1) 便是 每个中间件中的next
        // 所以在每个中间件执行中，遇到next 执行下一个中间件。那么这个中间件压入执行栈中
        // 所有中间件执行完，则中间件根据先进后出的顺序，一个个退出栈，形成洋葱结构。
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

假如有一个`mid`用来存放中间件：

```javascript
let mid = []

  function wait(par) {
      return new Promise((resolve, reject) => {
          setTimeout(() => {
            resolve(par)
          }, 100)
      })
  }

  mid.push(async (ctx, next) => {
    console.log(10)
    let w1 = await wait('1 waait')
    console.log('w1', w1)
    await next()
    console.log(11)
  })

  mid.push(async (ctx, next) => {
    console.log(20)
    let w2 = await wait('2 waait')
    console.log('w2', w2)
    await next()
    console.log(21)
  })

  mid.push(async (ctx, next) => {
    console.log(30)
    let w3 = await wait('3 waait')
    console.log('w3', w3)
    await next()
    console.log(31)
  })
```

执行：

```javascript
let gen = async () => {
    let res = await compose(mid)
    console.log(34444, res())
  }
  gen()
```

则可以看到输出：

```
10
34444 Promise { <pending> }
w1 1 waait
20
w2 2 waait
30
w3 3 waait
31
21
11
```