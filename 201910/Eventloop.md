# Eventloop

> 因为`javascript`是单线程，为了高效的执行产生了同步任务与异步任务，所以需要一个机制来调度不同的任务去执行。这个机制就是——`Eventloop`。

## 执行顺序

首先，我们先知道浏览器是按照以下顺序去执行的：

+ 1. 先执行当前同步环境代码。把异步任务分为宏任务与微任务，送进队列注册回调事件。

+ 2. 执行微任务队列的任务，直到微任务队列为空。

+ 3. 执行宏任务中到期的回调。

+ 4. 重复第二步。

...

### 宏任务/task

`setTimeout`、`setInterval`、`setImmediate`、`IO事件`、`UI交互`

### 微任务/microtask

`Promise`、`MutaionObserver`

除以上之外，还有一个`Process.nextTick`。这个很多地方把它归为微任务，但是注册时间是早于`Promise`的。这里可以把它理解为当前同步任务后执行。

看下一段代码：

```javascript
console.log(1)
setTimeout(() => {
	console.log(2)
	new Promise((reslove) => {
		console.log(7)
		reslove()
	}).then(() => {
		console.log(8)
	}).then(() => {
		console.log(9)
	})
}, 1000)

new Promise((reslove) => {
	console.log(3)
	reslove()
}).then(() => {
	console.log(4)
}).then(() => {
	console.log(5)
})

setTimeout(() => {
	console.log(6)
}, 2000)
```

> 所有涉及`Promise`的例子中, `then`中都是返回一个`Promise`。

无论在`node`还是`chrome`中都输出的是：`1,3,4,5,2,7,8,9,6`。

+ 先执行当前同步任务，输出`1、3`。同时在微任务队列中注册`Promise(4)`。

+ 执行当前微任务队列中的任务，输出`4`。同时注册微任务`Promise(5)`。由于微任务没有空，所以继续执行微任务，输出`5`。

+ 先执行当前同步任务，输出`2、7`。同时在微任务队列中注册`Promise(8)`。

+ 执行当前微任务队列中的任务，输出`8`。同时注册微任务`Promise(9)`。由于微任务没有空，所以继续执行微任务，输出`9`。

+ 再1秒后，执行`setTimeout`的回调，输出`6`

如果改成以下代码：

```javascript
console.log(1)
setTimeout(() => {
    console.log(2)
    new Promise((reslove) => {
        console.log(7)
        reslove()
    }).then(() => {
        console.log(8)
    }).then(() => {
        console.log(9)
    })
}, 1000)

new Promise((reslove) => {
    console.log(3)
    reslove()
}).then(() => {
    console.log(4)
}).then(() => {
    console.log(5)
})

setTimeout(() => {
    console.log(6)
    new Promise((reslove) => {
        console.log(10)
        reslove()
    }).then(() => {
        console.log(11)
    }).then(() => {
        console.log(12)
    })
}, 1000)
```

以上代码会在`chrome`中输出`1、3、4、5、2、7、8、9、6、10、11、12`，在node中会输出`1、3、4、5、2、7、6、10、8、11、9、12`。区别主要是两个`setTimeout`。

在`chrome`中：

+ 执行到第一个`setTimeout`的回调，输出`2、7`，同时在微任务中注册`Promsie(8)`

+ 执行当前微任务队列中的任务，输出`8`。同时注册微任务`Promise(9)`。由于微任务没有空，所以继续执行微任务，输出`9`。

+ 执行到第二个`setTimeout`的回调，输出`6、10`，同时在微任务中注册`Promsie(11)`

+ 执行当前微任务队列中的任务，输出`11`。同时注册微任务`Promise(12)`。由于微任务没有空，所以继续执行微任务，输出`12`。

说明在浏览器中，宏任务与微任务是严格间隔执行的，执行完完宏任务队列会立马执行微任务队列，直到微任务队列为空。再执行下一个宏任务。

在`node`中：

+ 执行到第一个`setTimeout`的回调，输出`2、7`，同时在微任务中注册`Promsie(8)`

+ 执行到第二个`setTimeout`的回调，输出`6、10`，同时在微任务中注册`Promsie(11)`

+ 执行微任务`Promsie(8)`，输出`8`，同时注册`Promise(9)`。微任务队列没空，继续执行`Promsie(11)`， 输出`11`，同时注册`Promise(12)`。继续执行下去，一次输出`9, 12`。

这说明`node`中，到期`timer`是在同时执行的，会把*所有到期的`timer`*都执行完，再执行微任务。说到这里，可以看node中的执行顺序了。


## node的Eventloop

在官方文档，可以了解每次循环有6个阶段。

+ `timers`。主要是指到期的`setTimeout/setInterval`。需要注意的是这里是根据到期的时间先后来执行的，并不存在一个队列。这个地方，从js的角度看怎么去定义timer到期是很好理解的。但是从`libuv`的源码上来看，是存在一个判断——若最小到期的timer时间大于`loop`运行时间，则表示没有到期的`timer`。至于什么是`loop`的运行时间，则苦于不会c/c++，无法吃透`libuv`的源码，也没有对应的资料。以后再明确。

+ `I/O callbacks/pending callbacks`。根据`libuv`文档，是指`poll`阶段因某些原因不能执行，被延迟到此轮执行的循环。根据`nodejs`文档，是除`timers, setImmediate, close`外的大部分回调都在此段执行。这里根据`poll`的理解，`libuv`文档的解释更合适些。这个地方执行的回到更准确的说是处理系统调用错误，比如`stream, pipe, tcp, udp通信的错误callback`。

+ `idle, prepare`。`nodejs`内部使用。具体执行什么也需要读`libuv`源码后才能知晓。

+ `poll`。这个阶段会阻塞等待监听基本上所有IO事件的回调。这个等待会有超时时间的，这个超时时间为最快到期的`timer`的时间。这个阶段会一直执行队列中的回调，直到队列为空或者达到执行回调的上限，则会进行下一个阶段。需要注意的是，如果`eventloop`每个阶段都为空，则会一直处于`poll`阶段，等待监听的回调，直到超时。

+ `check`。准确来说，这个阶段就只有一个`setTmmediate`。

+ `close`。主要是`socket.on(close)`。

+ `process.nextTick`。不太同于微任务，它会在以上任何一个阶段结束执行。

以上是宏任务，执行完后会执行微任务，直至队列为空。

## timer

主要是`setTimeout/setImmediate`的执行顺序。如上文说，一个是在`timer`阶段，一个是`check`阶段。咋看是`setTimeout`在前的。但是实际上我来看看。

```javascript
setTimeout(()=>{
	console.log(1)
})
setImmediate(() => {
	console.log(2)
})
```

它会在输出`1,2`与`2,1`间随机。

首先，我们得知道`setTimeout`的最小时间是1毫秒，也就是说当不传秒数或者小于1的时候，会默认为1。那么理论上`setImmediate`在前。但是实际情况上，如果`eventloop`准备时间大于1ms，那么`setTimeout`已到期，就会执行`timer`再执行`setImmediate`。如果小于1ms, 此时`timer`堆还为空，就会先执行`setImmediate`。

但是如果在`IO`回调中，他们的顺序是固定的。

```javascript
let fs = require('fs')
fs.readFile('./nn.html', () => {
	setTimeout(()=>{
		console.log(1)
	})
	setImmediate(() => {
		console.log(2)
	})
})
```

因为在读取文件也就是`IO`后，此时现处于`poll`阶段，那么之前立马就是`check`阶段，所以先会执行`setImmediate`。

从性能上来说，如果要做到异步执行，`setImmediate`比`setTimeout`要好的多。

最后需要注意的是遇到`await`是一个promise的语法糖.

```javaScript
function f() {
    await p()
    s()
}
// 等价于
function f() {
    resolve(p()).then(s())
}
```

```javaScript
console.log('script start')

async function async1() {
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2 end')
}
async1()

setTimeout(function() {
    console.log('setTimeout')
}, 0)

new Promise(resolve => {
    console.log('Promise')
    resolve()
})
    .then(function() {
    console.log('promise1')
    })
    .then(function() {
    console.log('promise2')
    })

console.log('script end')

// 输出script start
// 执行async1，
// 在async1 中执行async2，输出async2 end, 把async1 end放到微任务中
// settimeout放入延迟队列
// 输出promise，把primse1放入微任务
// 输出scrtip end
// 开始检查微任务，出书async1 end, primise1,放入promise2到微任务
// 输出promise2
// 输出setout
```

[一次弄懂Event Loop](https://zhuanlan.zhihu.com/p/55511602)
