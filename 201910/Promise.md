# Promise的自实现

## 定义

`Promise`主要是表达一个异步操作的结果或者是结果的承诺。结果有两种：`onFulfilled`与`onRejected`两种状态。

## 规范

现在主要流行的是(`Promise/A+`规范)[http://malcolmyu.github.io/malnote/2015/06/12/Promises-A-Plus/]，指定了符合`Prmoise`表现的方法、状态等。

#### 状态

+ Pending. 等待。

+ Fulfilled. 成功。

+ Rejected. 失败或者异常状态。

状态初始化时是`Pending`，可以转化为`Fulfilled`或`Rejected`。`Fulfilled`与`Rejected`不能互相转化。

+ 拥有`then`的方法。接受两个参数`onFulfilled`/`onRejected`，若不为函数可忽略。若为函数第一个参数为`Promise`的终值或者据因。`then`必须返回一个`promise`。

## 实现

> 本质上，我们可以根据`鸭子辩形`的思路来实现一个符合规范的`Promise`。

首先构造一个函数：

```javascript
// fn为Pro参数function
function Prom(fn) {
    // pro状态
    this.state = 'pending'
    // 值
    this.vaule = null
    // 回调函数存放
    this.deferred = []
    let that = this
    // 为Pro参数function中的reslove方法。
    // 用于触发回调。
    function reslove(newVal) {
        this.state = 'onFullfilled'
        this.value = newVal
        // 为了避免在then中函数触发前，执行reslove。
        setTimeout(() => {
            this.deferred.forEach(defer => {
                defer.call(that, newVal)
            })
        })
    }
    // 直接执行Pro参数函数，传参为reslove方法。
    fn(reslove.bind(this))
}
// 本质上 then就是一个收集Pro回调的地方，把回调放到一个数组中，
// 供reslove函数中一个个触发
Prom.prototype.then = function(onFullfilled) {
    this.deferred.push(onFullfilled)
}
```
以上只是实现了基础的`Promise`执行完异步后触发`then`的函数，如果要实现串行(多个then)，则就要做下改动。

```javascript
Prom.prototype.then = function(onFullfilled) {
    let that = this
    // 如果要实现串行，也就是必须同样返回一个`Pro`
    return new Prom((reslove) => {
        function handle(value) {
            // ************
            // 如果传参数是函数的话，如果有是一个promise（onFullfilled）的话 则需要执行then方法，把当前Promise的reslove（回调）放到onFullfilled的回调中，而onFullfilled会被that的回调触发。
            // *******
            var ret = (typeof onFullfilled === 'function') && onFullfilled(value) || value
            if (ret && typeof ret['then'] == 'function') {
                ret.then(function(value) {
                    reslove(value)
                })
            } else {
                reslove(ret)
            }
        }
        // 如果最开始的promise状态为pending
        if (that.state === 'pending') {
            that.deferred.push(handle)
        // 一般使用时都是走此分支
        } else {
            handle(that.value)
        }
    })
}

function Prom(fn) {
    this.state = 'pending'
    this.vaule = null
    this.deferred = []
    let that = this
    function reslove(newVal) {
        // ************
        // 如果是一个promise则，需要把当前回调放到promise的回调中执行
        / *******
        if (newVal && (typeof newVal == 'object' || typeof newVal == 'function')) {
            let then = newVal.then
            then(reslove)
            return
        } else {
            this.state = 'onFullfilled'
            this.value = newVal
            setTimeout(() => {
                this.deferred.forEach(defer => {
                    defer.call(that, newVal)
                })
            })
        }
    }
    fn(reslove.bind(this))
}
```

就拿以下代码流程举例：

```javascript
function p1() {
    return new Prom(function(resolve) {
    setTimeout(() => {
        console.log('0->1', this.index)
        resolve('p111')
    }, 1000)
    });
}
function p2(id) {
    return new Prom(function(resolve) {
    setTimeout(() => {
        console.log('0->2', this.index,  id)
        resolve('p222')
    }, 2000)
    });
}

p1().then(p2).then(function(res) {
    console.log('res-->', this, res)
})
```

1. 执行p1，返回一个`promise1`，此`promise1`拥有一个then方法，即`p1().then`。

2. 执行`promise1`的`then`，首先`then(p2)`会返回一个`promise2`,此拥有一个`then`方法， 即`(p2).then`。 需要特别注意的是，在生成`promise2`的同时，也执行到了`function p2(id)`的内部`promise`，返回一个`promise3`。此`promise3`相关函数会放到`promsie1`的回调中。<strong>(若到此时，根据后面流程，`promsie1`会触发的是`promsie3`，根本无法触发`promise2`，进而`promse2`的回到也无法执行。)</strong>此处根据代码的处理，会`promsie3`调用`promise2`的回调。

3. 执行`p2).then`其实就是把`res`放到`promise2`的回调集中。

4. 1秒后开始执行`promise1`的回调，进而执行`promise3`，再执行`promise2`的回调。

以下是`Prmise.all`的实现：

```JavaScript
Prom.all = function(promes) {
    // 参考then 肯定是要返回promsie的
    return new Prom(function(reslove) {
        var i = 0
        var reslut = []
        var len = promes.length
        var count = len
        function resolveAll(index, value) {
            reslut[index] = value
            if (--count == 0) {
                reslove(reslut)
            }
        }
        function re(index) {
            return function(value) {
                resolveAll(index, value)
            }
        }
        // 直接遍历promsie数组，因为需要用到index， 所以re函数包一层形成闭包
        for(; i < len; i++) {
            promes[i].then(re(i))
        }
    })
}
```

`Prmise.trace`：

```javascript
Prom.trace = function(promes) {
    return new Prom(function(reslove) {
        var i = 0;
        var len = promes.length;
        for(; i < len; i++) {
            promes[i].then(function(value) {
                reslove(value)
            })
        }
    })
}
```

[参考](https://juejin.im/post/5b2f02cd5188252b937548ab#heading-6)
