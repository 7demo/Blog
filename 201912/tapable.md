# tapable

> tapable 是 webpack的核心类库，它可以挂载钩子、触发钩子。就是一个观察消费者订阅器。

## 用法：

### SyncHook

根据文档，`tapable`有很多hook，hook可以创建钩子。

```javaScript
const {
	SyncHook,
	SyncBailHook,
	SyncWaterfallHook,
	SyncLoopHook,
	AsyncParallelHook,
	AsyncParallelBailHook,
	AsyncSeriesHook,
	AsyncSeriesBailHook,
	AsyncSeriesWaterfallHook
 } = require("tapable")
```

先看最简单的`SyncHook`，是创建同步钩子。

```javaScript
const hook = new SyncHook(['speed1', 'speed2'])
hook.tap('hook1', (...arg) => {
    console.log('syncHook1', arg)
})
hook.tap('hook2', (...arg) => {
    console.log('syncHook2', arg)
})
```

以上通过`hook.call(1,2,3)`触发钩子后，将会输出：`syncHook1 [ 1, 2 ]、syncHook2 [ 1, 2 ]`，说明：

1, 实例`SyncHook`时，参数个数直接决定了调用时可以接受的个数，以上例子，实参中第三个值并未被接收。注意这个形参必须为非空字符串。

2，可以压入多个钩子，并且被调用时也是按照注册顺序。

### SyncBailHook

区别在于，如果挂载的钩子，如果返回值为非`undeifined`，则后续的钩子则不会执行。

### SyncWaterfallHook

则是每个钩子的返回值，则会传递给下个钩子

### SyncLoopHook

同步钩子，如果挂载的函数没有返回undefined，则会一直执行直至返回undefined。

### AsyncParallelHook

可以挂载异步钩子，需要用callAsync调用

```javaScript
const hook = new AsyncParallelHook()
hook.tapAsync('hook1', (cb) => {
    setTimeout(() => {
        console.log('syncHook1', cb)
        cb()
    }, 1000)
    console.log('hook1')
})
hook.tapAsync('hook2', (cb) => {
    setTimeout(() => {
        console.log('syncHook2', cb)
        cb()
    })
})
hook.callAsync(() => {
    console.log('final')
})
```

以上会输出:

```javaScript
hook1
syncHook2 _err1 => {
if(_err1) {
if(_counter > 0) {
_callback(_err1);
_counter = 0;
}
} else {
if(--_counter === 0) _done();
}
}
syncHook1 _err0 => {
if(_err0) {
if(_counter > 0) {
_callback(_err0);
_counter = 0;
}
} else {
if(--_counter === 0) _done();
}
}
final
```

可以看出来：是按照钩子中函数顺序同步执行的，每个钩子函数都会有一个`callback`，执行完则会执行传的回调函数。

当时钩子中挂载的是返回的`promise`，则需要使用`tapPromise/callPromise`

其他方法则与同步方类似。除此之外，还可以添加拦截器：

```javaScript
hook.intercept({
    call: (...arg)=> {
    console.log('call--->', arg)
    },
    register: (arg) => {
    console.log('register----->', arg)
    return arg
    },
    tap(...arg){
    console.log('tap---->', arg)
    },
    loop(...arg){
    console.log('loop---->', arg)
    }
})

// output
/*
register-----> { type: 'sync', fn: [Function], name: 'hook1' }
register-----> { type: 'sync', fn: [Function], name: 'hook2' }
call---> [ 1, 2 ]
tap----> [ { type: 'sync', fn: [Function], name: 'hook1' } ]
syncHook1 [ 1, 2 ]
tap----> [ { type: 'sync', fn: [Function], name: 'hook2' } ]
syncHook2 [ 1, 2 ]
*/
```
1，register 是每挂载一个钩子时触发

2, call是钩子触发前执行一次

3，tap每个钩子触发前都会执行。其实loop时也会执行。

4，loop每次loop都会执行


## 源码

整个入口文件是`lib/index.js`:

```javaScript
exports.__esModule = true;
exports.SyncHook = require("./SyncHook");
exports.SyncBailHook = require("./SyncBailHook");
exports.SyncWaterfallHook = require("./SyncWaterfallHook");
exports.SyncLoopHook = require("./SyncLoopHook");
exports.AsyncParallelHook = require("./AsyncParallelHook");
exports.AsyncParallelBailHook = require("./AsyncParallelBailHook");
exports.AsyncSeriesHook = require("./AsyncSeriesHook");
exports.AsyncSeriesBailHook = require("./AsyncSeriesBailHook");
exports.AsyncSeriesLoopHook = require("./AsyncSeriesLoopHook");
exports.AsyncSeriesWaterfallHook = require("./AsyncSeriesWaterfallHook");
exports.HookMap = require("./HookMap");
exports.MultiHook = require("./MultiHook");
```

可以看出来，`tapable`就是多个钩子方法而已。直接看`SyncHook.js`

```javaScript
"use strict";

const Hook = require("./Hook");
const HookCodeFactory = require("./HookCodeFactory");

class SyncHookCodeFactory extends HookCodeFactory {
	content({ onError, onDone, rethrowIfPossible }) {
		return this.callTapsSeries({
			onError: (i, err) => onError(err),
			onDone,
			rethrowIfPossible
		});
	}
}

const factory = new SyncHookCodeFactory();

const TAP_ASYNC = () => {
	throw new Error("tapAsync is not supported on a SyncHook");
};

const TAP_PROMISE = () => {
	throw new Error("tapPromise is not supported on a SyncHook");
};

const COMPILE = function(options) {
	factory.setup(this, options);
	return factory.create(options);
};

function SyncHook(args = [], name = undefined) {
	const hook = new Hook(args, name);
	hook.constructor = SyncHook;
	hook.tapAsync = TAP_ASYNC;
	hook.tapPromise = TAP_PROMISE;
	hook.compile = COMPILE;
	return hook;
}

SyncHook.prototype = null;

module.exports = SyncHook;
```

以上代码可以看出，主要做了两件事：

1, `SyncHook`主要是`Hook`类的实例返回的也是这个实例。

2，对`tagAsync/tapPromise/Compile`进行重写。其中，`tapAsync/tagPromise`从使用方法上就知道，是`Synchook`不支持。`compile`待下文说。


再先看下`Hook.js`:


```javaScript

class Hook {
	constructor(args = [], name = undefined) {
		this._args = args;
		this.name = name;
		this.taps = [];
		this.interceptors = [];
		this._call = CALL_DELEGATE;
		this.call = CALL_DELEGATE;
		this._callAsync = CALL_ASYNC_DELEGATE;
		this.callAsync = CALL_ASYNC_DELEGATE;
		this._promise = PROMISE_DELEGATE;
		this.promise = PROMISE_DELEGATE;
		this._x = undefined;

		this.compile = this.compile;
		this.tap = this.tap;
		this.tapAsync = this.tapAsync;
		this.tapPromise = this.tapPromise;
	}

    // 等待被重写，其实就是编译
	compile(options) {
		throw new Error("Abstract: should be overridden");
	}

	_createCall(type) {
		return this.compile({
			taps: this.taps,
			interceptors: this.interceptors,
			args: this._args,
			type: type
		});
	}

	_tap(type, options, fn) {
		if (typeof options === "string") {
			options = {
				name: options
			};
		} else if (typeof options !== "object" || options === null) {
			throw new Error("Invalid tap options");
		}
		if (typeof options.name !== "string" || options.name === "") {
			throw new Error("Missing name for tap");
		}
		if (typeof options.context !== "undefined") {
			deprecateContext();
		}
		options = Object.assign({ type, fn }, options);
		options = this._runRegisterInterceptors(options);
		this._insert(options);
	}

	tap(options, fn) {
		this._tap("sync", options, fn);
	}

	tapAsync(options, fn) {
		this._tap("async", options, fn);
	}

	tapPromise(options, fn) {
		this._tap("promise", options, fn);
    }
    // ...
}

Object.setPrototypeOf(Hook.prototype, null);

module.exports = Hook;

```

在实例`SyncHook`时`const hook = new SyncHook(['speed1', 'speed2'])`，就是一个`Hook`的实例。

```javaScript
Hook: {
    this.name = name;
    // 注入的函数
    this.taps = [];
    // 存放拦截器
    this.interceptors = [];
    this._call = CALL_DELEGATE;
    this.call = CALL_DELEGATE;
    this._x = undefined;
    this.compile = this.compile;
    this.tap = this.tap;
}
```

在挂载函数时, 使用tab：

```javaScript
hook.tap('hook1', (...arg) => {
    console.log('syncHook1', arg)
})
```

此时是`hook`调用了`_tap`

```javaScript
tap(options, fn) {
    // opttions 是 hook1
    // fn 是 （)=>{}
    this._tap("sync", options, fn);
}
_tap(type, options, fn) {
    // 其实是钩子函数名称转为对象
    if (typeof options === "string") {
        options = {
            name: options
        };
    }
    /**
     * type: sync
     * fn: () => {}
     * name: hook1
     */
    options = Object.assign({ type, fn }, options);
    options = this._runRegisterInterceptors(options);
    this._insert(options);
}
/**
 * 判断是否已注入拦截器，
 */
_runRegisterInterceptors(options) {
    for (const interceptor of this.interceptors) {
        if (interceptor.register) {
            const newOptions = interceptor.register(options);
            if (newOptions !== undefined) {
                options = newOptions;
            }
        }
    }
    return options;
}
```

最后一步，调用`_insert`插入，就是把options放入`this.taps中`。

```javaScript
_insert(item) {
    this._resetCompilation();
    let before;
    if (typeof item.before === "string") {
        before = new Set([item.before]);
    } else if (Array.isArray(item.before)) {
        before = new Set(item.before);
    }
    let stage = 0;
    if (typeof item.stage === "number") {
        stage = item.stage;
    }
    let i = this.taps.length;
    while (i > 0) {
        i--;
        const x = this.taps[i];
        this.taps[i + 1] = x;
        const xStage = x.stage || 0;
        if (before) {
            if (before.has(x.name)) {
                before.delete(x.name);
                continue;
            }
            if (before.size > 0) {
                continue;
            }
        }
        if (xStage > stage) {
            continue;
        }
        i++;
        break;
    }
    this.taps[i] = item;
}
```

到现在，把两个函数都放进`taps`数组中了。

如果我们再挂载拦截器：

```javaScript
hook.intercept({
    call: (...arg)=> {
    console.log('call--->', arg)
    },
    register: (arg) => {
    console.log('register----->', arg)
    return arg
    },
    tap(...arg){
    console.log('tap---->', arg)
    },
    loop(...arg){
    console.log('loop---->', arg)
    }
})
```

就会执行`hook.intercept`:

```javaScript
intercept(interceptor) {
    this._resetCompilation();
    this.interceptors.push(Object.assign({}, interceptor));
    if (interceptor.register) {
        for (let i = 0; i < this.taps.length; i++) {
            // 根据挂载的钩子函数 直接执行一次register 并且返回tap
            this.taps[i] = interceptor.register(this.taps[i]);
        }
    }
}
```

此时执行`hook.call(1,2,3)`,其实是执行了:

```javaScript
const CALL_DELEGATE = function(...args) {
    // args是`[1,2,3]`
	this.call = this._createCall("sync");
	return this.call(...args);
};
_createCall(type) {
    return this.compile({
        // 两个钩子函数
        taps: this.taps,
        // 拦截器
        interceptors: this.interceptors,
        // [speed1, speed2]
        args: this._args,
        // sync
        type: type
    });
}
```

到最后执行了`compile`。

```javaScript
class SyncHookCodeFactory extends HookCodeFactory {
	content({ onError, onDone, rethrowIfPossible }) {
		return this.callTapsSeries({
			onError: (i, err) => onError(err),
			onDone,
			rethrowIfPossible
		});
	}
}

const factory = new SyncHookCodeFactory();
const COMPILE = function(options) {
    // 这个this是执行时的this，即Hook
	factory.setup(this, options);
	return factory.create(options);
};

```

`compile`是`SyncHookCodeFactory`的一个实例，它又是基于`HookCodeFactory`继承实现了。

```javaScript
setup(instance, options) {
    instance._x = options.taps.map(t => t.fn);
}
```

就是对`Hook._x`进行了赋值，值为包含先后注入的函数`hoo1,hook2`的对象

```javaScript
// 最终返回一个fn， new Function()
create(options) {
    this.init(options);
    let fn;
    switch (this.options.type) {
        case "sync":
            /**
             * 先看最后返回如此
             * /**
            * 此时call函数应该为:
            * "use strict";
            * // 参数为this.args 返回
            * function (speed1, speed2) {
            * // 这是header（）返回
            *  var _context;
            *  var _x = this._x;
            *  var _taps = this.taps;
            *  var _interceptors = this.interceptors;
            *  _interceptors[0].call(speed1, speed2);
            *  // 这是content返回
            *  "var _loop;
                do {
                    _loop = false;
                    _interceptors[0].loop(speed1, speed2);
                    var _tap0 = _taps[0];
                    _interceptors[0].tap(_tap0);
                    var _fn0 = _x[0];
                    var _result0 = _fn0(speed1, speed2);
                    if(_result0 !== undefined) {
                        _loop = true;
                    } else {
                        var _tap1 = _taps[1];
                        _interceptors[0].tap(_tap1);
                        var _fn1 = _x[1];
                        var _result1 = _fn1(speed1, speed2);
                        if(_result1 !== undefined) {
                            _loop = true;
                        } else {
                            if(!_loop) {
                            }
                        }
                    }
                } while(_loop);
                "
            *}
            */
            fn = new Function(
                this.args(),
                '"use strict";\n' +
                    this.header() +
                    this.content({
                        onError: err => `throw ${err};\n`,
                        onResult: result => `return ${result};\n`,
                        resultReturns: true,
                        onDone: () => "",
                        rethrowIfPossible: true
                    })
            );
            break;
    }
    this.deinit();
    return fn;
}
```

最终，返回的函数如下，形参为speed1, speed2。此即为`call`函数。在执行`call(1,2,3)`时，则只会接受两个前两个参数。

```javaScript
function anonymous(speed1, speed2
) {
    "use strict";
    var _context;
    // 一个数组，分别为通过tap挂的钩子方法
    var _x = this._x;
    // 挂载的钩子函数
    var _taps = this.taps;
    var _interceptors = this.interceptors;
    // 执行第一个拦截器的call方法，也就是说所有钩子执行前执行一次call
    _interceptors[0].call(speed1, speed2);
    var _loop;
    do {
        _loop = false;
        var _tap0 = _taps[0];
        // 每个钩子执行前执行一次tap
        _interceptors[0].tap(_tap0);
        var _fn0 = _x[0];
        var _result0 = _fn0(speed1, speed2);
        // 如果不是返回undefined就一直执行下去
        if(_result0 !== undefined) {
            _loop = true;
        } else {
            // 如果返回的是undefined 则就继续执行第二个函数，这里直接把逻辑关系给套进去了
            var _tap1 = _taps[1];
            _interceptors[0].tap(_tap1);
            var _fn1 = _x[1];
            var _result1 = _fn1(speed1, speed2);
            if(_result1 !== undefined) {
                _loop = true;
            } else {
                if(!_loop) {
                }
            }
        }
    } while(_loop);
}
```

最终的call代码逻辑明晰后就要去看下 三个关键的地方`this.args`, `this.headers`, `this.content`

#### this.args

```javaScript
args({ before, after } = {}) {
    let allArgs = this._args;
    if (before) allArgs = [before].concat(allArgs);
    if (after) allArgs = allArgs.concat(after);
    if (allArgs.length === 0) {
        return "";
    } else {
        return allArgs.join(", ");
    }
}
```

最终返回了字符串分割的形参

#### this.header

```javaScript
// 挂载的钩子是否有context属性，其实就是上下文
needContext() {
    for (const tap of this.options.taps) if (tap.context) return true;
    return false;
}
header() {
    let code = "";
    if (this.needContext()) {
        code += "var _context = {};\n";
    } else {
        code += "var _context;\n";
    }
    // 拿到形参
    code += "var _x = this._x;\n";
    // 如果有拦截器的话，则把拦截器放进
    if (this.options.interceptors.length > 0) {
        code += "var _taps = this.taps;\n";
        code += "var _interceptors = this.interceptors;\n";
    }
    // 如果拦截器有call，则调用，如果有上下文context，则实参会依赖上下文
    for (let i = 0; i < this.options.interceptors.length; i++) {
        const interceptor = this.options.interceptors[i];
        if (interceptor.call) {
            code += `${this.getInterceptor(i)}.call(${this.args({
                before: interceptor.context ? "_context" : undefined
            })});\n`;
        }
    }
    return code;
}
```

#### this.content

```javaScript
this.content({
    onError: err => `throw ${err};\n`,
    onResult: result => `return ${result};\n`,
    resultReturns: true,
    onDone: () => "",
    rethrowIfPossible: true
})
```

`content`是构造函数时声明的：

```javaScript
class SyncHookCodeFactory extends HookCodeFactory {
    content({ onError, onDone, rethrowIfPossible }) {
        return this.callTapsSeries({
            onError: (i, err) => onError(err),
            onDone,
            rethrowIfPossible
        });
    }
}
```

找到了`callTapsSeries`

```javaScript
callTapsSeries({
    onError,
    onResult,
    resultReturns,
    onDone,
    doneReturns,
    rethrowIfPossible
}) {
    // 如果钩子为空 则直接执行结束函数
    if (this.options.taps.length === 0) return onDone();
    // 返回第一个异步钩子的下标 否则为-1
    const firstAsync = this.options.taps.findIndex(t => t.type !== "sync");
    // true
    const somethingReturns = resultReturns || doneReturns || false;
    let code = "";
    // 当前函数，默认为整个结束函数
    let current = onDone;
    for (let j = this.options.taps.length - 1; j >= 0; j--) {
        const i = j;
        // 如果是异步函数切当前函数等于结束函数，就是最后一个了
        // 则声明一个_next函数 函数内部为 结束函数
        // 最后返回这个_next函数
        const unroll = current !== onDone && this.options.taps[i].type !== "sync";
        if (unroll) {
            code += `function _next${i}() {\n`;
            code += current();
            code += `}\n`;
            current = () => `${somethingReturns ? "return " : ""}_next${i}();\n`;
        }
        const done = current;
        const doneBreak = skipDone => {
            if (skipDone) return "";
            return onDone();
        };
        // 使用的为callTap返回内容
        const content = this.callTap(i, {
            onError: error => onError(i, error, done, doneBreak),
            onResult:
                onResult &&
                (result => {
                    return onResult(i, result, done, doneBreak);
                }),
            onDone: !onResult && done,
            rethrowIfPossible:
                rethrowIfPossible && (firstAsync < 0 || i < firstAsync)
        });
        current = () => content;
    }
    code += current();
    return code;
}
callTap(tapIndex, { onError, onResult, onDone, rethrowIfPossible }) {
    let code = "";
    // 标志是否缓存
    let hasTapCached = false;
    for (let i = 0; i < this.options.interceptors.length; i++) {
        const interceptor = this.options.interceptors[i];
        if (interceptor.tap) {
            if (!hasTapCached) {
                // 拿到钩子函数
                code += `var _tap${tapIndex} = ${this.getTap(tapIndex)};\n`;
                hasTapCached = true;
            }
            // 生成拦截器中tap方法代码
            code += `${this.getInterceptor(i)}.tap(${
                interceptor.context ? "_context, " : ""
            }_tap${tapIndex});\n`;
        }
    }
    // 拿到对应的钩子函数
    code += `var _fn${tapIndex} = ${this.getTapFn(tapIndex)};\n`;
    const tap = this.options.taps[tapIndex];
    switch (tap.type) {
        case "sync":
            if (!rethrowIfPossible) {
                code += `var _hasError${tapIndex} = false;\n`;
                code += "try {\n";
            }
            // 执行该钩子拿到返回值
            if (onResult) {
                code += `var _result${tapIndex} = _fn${tapIndex}(${this.args({
                    before: tap.context ? "_context" : undefined
                })});\n`;
            } else {
                code += `_fn${tapIndex}(${this.args({
                    before: tap.context ? "_context" : undefined
                })});\n`;
            }
            if (!rethrowIfPossible) {
                code += "} catch(_err) {\n";
                code += `_hasError${tapIndex} = true;\n`;
                code += onError("_err");
                code += "}\n";
                code += `if(!_hasError${tapIndex}) {\n`;
            }
            if (onResult) {
                code += onResult(`_result${tapIndex}`);
            }
            if (onDone) {
                code += onDone();
            }
            if (!rethrowIfPossible) {
                code += "}\n";
            }
            break;
    }
    return code;
}
```

以上就是`SyncHook`的流程，其他的则都是类似。

参考：[源码学习记录: tapable](https://segmentfault.com/a/1190000017421077?utm_source=tag-newest)
