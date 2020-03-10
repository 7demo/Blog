# Co

正常情况下，`generator`函数需要手动调用`next`方法才能执行：

```javaScript
let fs = require('fs')
var readFile = function(file) {
	return new Promise(function(reslove, reject) {
		fs.readFile(file, function(error, data) {
			if (error) {
				reject(error)
			}
			reslove(data)
		})
	})
}

var g = function* () {
	var f1 = yield readFile('./nn.html')
	var f2 = yield readFile('./PLUS楼层商品4.png')
	console.log('f1--->', f1)
	console.log('f2--->', f2)
}

var gen = g()

console.log(gen.next())
console.log('---')
console.log(gen.next())
console.log('---')
console.log(gen.next())
```

依次会输出：

```
{ value: Promise { <pending> }, done: false }
---
{ value: Promise { <pending> }, done: false }
---
f1---> undefined
f2---> undefined
{ value: undefined, done: true }
```

需要显式的调用`gen.next()`，每次返回一个对象包含`value`与`done`。如果能自动执行则是：

```javaScript
gen.next().value.then(function(data) {
    gen.next().value.then(function(data) {
        g.next(data)
    })
})
```

所以可以实现一个方法，能让`generation`自动执行：

```javaScript
function run(g) {
	var gen = g()
	function next(data) {
		let ret = gen.next(data)
		if (ret.done) {
			return ret.value
		}
		ret.value.then(function(data) {
			next(data)
		})
	}
	next()
}
run(g)
```

`Co.js`的源码也如此：

```javaScript
function co(gen) {
    var ctx = this;
    var args = slice.call(arguments, 1);
    return new Promise(function(resolve, reject) {
      if (typeof gen === 'function') gen = gen.apply(ctx, args);
      if (!gen || typeof gen.next !== 'function') return resolve(gen);

      onFulfilled();

      /**
       * @param {Mixed} res
       * @return {Promise}
       * @api private
       */

      function onFulfilled(res) {
        var ret;
        try {
          ret = gen.next(res);
        } catch (e) {
          return reject(e);
        }
        next(ret);
        return null;
      }

      /**
       * @param {Error} err
       * @return {Promise}
       * @api private
       */

      function onRejected(err) {
        var ret;
        try {
          ret = gen.throw(err);
        } catch (e) {
          return reject(e);
        }
        next(ret);
      }

      /**
       * Get the next value in the generator,
       * return a promise.
       *
       * @param {Object} ret
       * @return {Promise}
       * @api private
       */

      function next(ret) {
        if (ret.done) return resolve(ret.value);
        var value = toPromise.call(ctx, ret.value);
        if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
        return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
          + 'but the following object was passed: "' + String(ret.value) + '"'));
      }
    });
  }
```

### Thunkify

```javaScript
function thunkify(fn) {
	let ctx = this
	return function() {
		let args = Array.from(arguments)
		return function(cb) {
			args.push(cb)
			fn.apply(ctx, args)
		}
	}
}
```

调用

```javaScrit
var f = thunkify(fs.readFile)
f('./nn.html')(function(error, data) {
	console.log(error, data)
})
```

当然以上代码存在回调函数多次调用的问题，但是具体场景待发现。

### async实现

> async是gengerator的语法糖。主要区别是内置执行器与返回promise。它的实现也是自执行函数与generator包再一个函数中。

```javaScript
async fn(args){}

// 等同

function fn(args) {
  return spawn(function* () {
    // xx
  })
}

function spawn(gen) {
  return new Promise((resolve, reject) => {
    let g = gen()
    function step(value) {
      let next = value()
      if (next.done) {
        return reslove(next.value)
      }
      Promise.reslove(next.value).then((data) => {
        step(function() {
          return g.next(data)
        })
      })
    }
    step(function() {
      return g.next()
    })
  })
}
```
