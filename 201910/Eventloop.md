# Eventloop

> 因为`javascript`是单线程，产生了同步任务与异步任务，所以需要一个机制来调度不同的任务去执行。这个机制就是——`Eventloop`。

## 执行顺序

首先，我们先不做区分执行环境的这样去定义执行顺序：

+ 1. 先执行当前同步环境代码。把异步任务分为宏任务与微任务，送进队列注册回调事件。

+ 2. 执行微任务队列的任务，直到微任务队列为空。

+ 3. 执行宏任务中到期的回调。

+ 4. 重复第二步。

...

### 宏任务/task

`setTimeout`、`setInterval`、`setImmediate`、`IO事件`、`UI交互`

### 微任务/microtask

`Promise`、`MutaionObserver`

除以上之外，还有一个`Process.nexttick`。这个很多地方把它归为微任务，但是注册时间是早于`Promise`的。这里可以把它理解为当前同步任务后执行。

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

说明在浏览器中，宏任务与微任务是严格间隔执行的，执行完一个宏任务会立马执行微任务队列，直到微任务队列为空。再执行下一个宏任务。

在`node`中：

+ 执行到第一个`setTimeout`的回调，输出`2、7`，同时在微任务中注册`Promsie(8)`

+ 执行到第二个`setTimeout`的回调，输出`6、10`，同时在微任务中注册`Promsie(11)`

+ 执行微任务`Promsie(8)`，输出`8`，同时注册`Promise(9)`。微任务队列没空，继续执行`Promsie(11)`， 输出`11`，同时注册`Promise(12)`。继续执行下去，一次输出`9, 12`。

这说明`node`中，到期`timer`是在同时执行的，会把*所有到期的`timer`*都执行完，再执行微任务。

## timer

主要是`setTimeout/setInterval/setImmediate`。

