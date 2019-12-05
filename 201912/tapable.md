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
 * 如果第一次挂载，则返回的是原options
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

就是对`Hook._x`进行了赋值，值为先后注入的函数`hoo1,hook2`。

```javaScript
create(options) {
    this.init(options);
    let fn;
    switch (this.options.type) {
        case "sync":
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
init(options) {
    this.options = options;
    this._args = options.args.slice();
}
deinit() {
    this.options = undefined;
    this._args = undefined;
}
header() {
    let code = "";
    if (this.needContext()) {
        code += "var _context = {};\n";
    } else {
        code += "var _context;\n";
    }
    code += "var _x = this._x;\n";
    if (this.options.interceptors.length > 0) {
        code += "var _taps = this.taps;\n";
        code += "var _interceptors = this.interceptors;\n";
    }
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
