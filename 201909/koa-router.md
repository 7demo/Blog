# koa-router

在`koa`中路由的本质就是在中间件中，通过`ctx.url`匹配地址进行处理。类似如下代码：

```javascript
app.use((ctx, next) => {
    switch ctx.url:
        case '/'
            ctx.body = 'index'
            break;
        case '/body':
            ctx.body = 'body'
            break;
})
```

但是这样存在着代码繁琐，无法匹配动态路由等问题。不过`koa-router`也是基于此原理来实现的。

`koa-router`源码中有两个文件`router.js`与`layer.js`，分别对应两个对象：`Router`与`Layer`。`Layer`主要是单个路由对象，它拥有路径匹配规则、参数获取、中间件等。`Router`主要是对外暴露的方法，是一个构造函数，用来在`koa`中接受路由注册，并且在`koa`的中间件中进行匹配处理。

这之前首先要简单了解一下`path-to-regexp`，它用来处理路径，返回对应的匹配规则。

### Router

`Router`作为一个构造函数：

```javascript
// router.js
var compose = require('koa-compose)
var methods = require('methods')

function Router(opts) {
    // 用来在引用时  去 关键字 new
    // 即，const router = require('koa-router')()
    // 与 const router = new require('koa-router)()
    if (!(this instanceof Router)) {
        return new Router(opts)
    }
    this.opts = opts || {}
    this.methods = this.opts.methods || [
        'HEAD',
        'OPTIONS',
        'GET',
        'PUT',
        'PATCH',
        'POST',
        'DELETE'
    ]
    // 路由参数的中间件组成的对象
    this.params = {}
    // 存放单个注册路由layer
    this.stack = []
}
```
我们使用`router.get/post`等方式挂载路由，所以要根据`methods`遍历，进而在`Router`的原型链上实现对应的方法。

```
methods.forEach(function(method) {
    Router.prototype[method] = function (name, path, middleware) {
        var middleware
        // 如果第二个参数是字符串或者是正则表达式，则第一个参数是name。否则第二个参数是函数，则第一个参数name其实就是path
        if (typeof path === 'string' || path instanceof RegExp) {
            middleware = Array.prototype.slice(arguments, 2)
        } else {
            middleware = Array.prototype.slice.call(arguments, 1)
            path = name
            name = null
        }
        this.register(path, [method], middleware, {
            name: name
        })
        // 方便链式调用
        return this
    }
})
```

挂载路由时，其实就是根据参数进行注册路由。

```javascript
Router.prototype.register = function(path, methods, middleware, opts) {
    opts = opts || {}
    var router = this
    var stack = this.stack

    // 如果路径是一个数组，则需要为每个独立注册
    if (Array.isArray(path)) {
        path.forEach(function(p) {
            router.register.call(router, p, methods, middleware, opts)
        })
        return this
    }

    // 生成单个layer路由对象
    var route = new Layer(path, methods, middleware, {
        end: opts.end === false ? opts.end : true,
        name: opts.name,
        sensitive: opts.sensitive || this.opts.sensitive || false,
        strict: opts.strict || this.opts.strict || false,
        prefix: opts.prefix || this.opts.prefix || '',
        ignoreCaptures: opts.ignoreCaptures
    })

    // 前缀处理
    if (this.opts.prefix) {
        router.setPrefix(this.opts.prefix)
    }

    // 
    Object.keys(this.params).forEach(function(param) {
        route.param(param, this.params[param])
    }, this)

    // 把单个路由对象放入stack中
    stack.push(route)
    
    return route

}
```

以上，通过`register`其实就是创建单个路由对象`larer`，存到`stack`中。

我们再看下`layer.js`,这里贴出整个代码：

```javascript
var pathToRegExp = require('path-to-regexp');
var uri = require('urijs');

function Layer(path, methods, middleware, opts) {
    this.opts = opts || {}
    this.name = this.opts.name || null
    // 该路由支持的方法
    this.methods = []
    this.paramNames = []
    this.stack = Array.isArray(middleware) ? middleware : [middleware]

    // 把此layer的方法都放入method中，并且HEAD同get的请求同样
    methods.forEach(function (method) {
        var l = this.methods.push(method.toUpperCase())
        if (this.methods[l - 1] === 'GET') {
            this.methods.unshift('HEAD')
        }
    }, this)

    // 保证中间件都是函数
    this.stack.forEach(function(fn) {
        var type = (typeof fn);
        if (type !== 'function') {
        throw new Error(
            methods.toString() + " `" + (this.opts.name || path) +"`: `middleware` "
            + "must be a function, not `" + type + "`"
        );
        }
    }, this);

    this.path = path
    // 根据路径，参数获得该路由的正则表达式
    this.regexp = pathToRegExp(path, this.paramNames, this.opts)

}

// 扩展match方法，来检测该layer是否匹配path
Layer.prototype.match = function(path) {
    return this.regexp.test(path)
}

//根据path，返回路径参数key-value对象
Layer.prototype.params = function(path, captures, existingParams) { 
    var params = existingParams || {}
    for (var len = captures.length, i=0; i<len; i++) {
        if (this.paramNames[i]) {
            var c = captures[i]
            params[this.paramNames[i].name] = c ? safeDecodeURICompoentent(c) : c
        }
    }
    return params
}

// 返回一个路径的数组，包含匹配的路径
Layer.prototype.captures = function(path) {
    if (this.opts.ignoreCaptures) {
        return []
    }
    return path.match(this.regexp).slice(1)
}

// 根据参数路径，产生地址
Layer.prototype.url = function(params, options) {
    var args = params
    var url = this.path.replace(/\(\.\*\)/g, '')
    var toPath = pathToRegExp.compile(url)
    var replaced
    if (typeof params != 'object') {
        args = Array.prototype.slice.call(arguments)
        if (typeof args[args.length - 1] == 'object') {
            options = args[args,length - 1]
            args = args.slice(0, args.length - 1)
        }
    }
    var tokens = pathToRegExp.parse(url)
    var replace = {}

    if (args instanceof Array) {
        for (var len = tokens.length, i = 0, j = 0; i < len; i++) { 
            if (tokens[i].name) replace[tokens[i].name] = args[j++]
        }
    } else if (tokens.some(token => token.name)) {
        replace = params
    } else {
        options = params
    }
    replaced = toPath(replaced)
    if (options && options.query) {
        var replaced = new uri(replaced)
        replaced.search(options.query)
        return replaced.toString()
    }
    return replaced
}

// 获取路由params参数键值对
Layer.prototype.param = function(param, fn) {
    var stack = this.stack
    var params = this.paramNames
    var middleware = function(ctx, next) {
        return fn.call(this, ctx.params[param], ctx, next)
    }
    middleware.param = param

    var names = params.map(function(p) {
        return p.name
    })

    var x = names.indexOf(param)

    if (x > 1) {
        stack.some(function(fn, i) {
            if (!fn.param || names.indexOf(fn.param) > x) {
                stack.splice(i, 0, middleware)
                return true
            }
        })
    }
    return this
}

Layer.prototype.setPrefix = function(prefix) {
    if (this.path) {
        this.path = prefix + this.path
        this.paramNames = []
        this.regexp = pathToRegExp(this.path, this.paramNames, this.opts）
    }
    return this
}

function safeDecodeURIComponent(text) {
  try {
    return decodeURIComponent(text);
  } catch (e) {
    return text;
  }
}

```

当然我们采用`router.get('/test')`形式把所有单个路由实例`layer`放到实例`router`的`stack`中。然后扩展`Router`原型链，增加一个`routes`方法，返回一个中间件，此中间件放到`koa`中间件栈中，共请求的时候调用。

```javascript
Router.prototype.routes = Router.prototype.middleware = function () {
  var router = this;

  var dispatch = function dispatch(ctx, next) {
    debug('%s %s', ctx.method, ctx.path);

    var path = router.opts.routerPath || ctx.routerPath || ctx.path;
    var matched = router.match(path, ctx.method);
    var layerChain, layer, i;

    if (ctx.matched) {
      ctx.matched.push.apply(ctx.matched, matched.path);
    } else {
      ctx.matched = matched.path;
    }

    ctx.router = router;

    if (!matched.route) return next();

    var matchedLayers = matched.pathAndMethod
    var mostSpecificLayer = matchedLayers[matchedLayers.length - 1]
    ctx._matchedRoute = mostSpecificLayer.path;
    if (mostSpecificLayer.name) {
      ctx._matchedRouteName = mostSpecificLayer.name;
    }
    // 把单个路由上的N个中间件处理，匹配路由时进行处理。
    layerChain = matchedLayers.reduce(function(memo, layer) {
      memo.push(function(ctx, next) {
        ctx.captures = layer.captures(path, ctx.captures);
        ctx.params = layer.params(path, ctx.captures, ctx.params);
        ctx.routerName = layer.name;
        return next();
      });
      return memo.concat(layer.stack);
    }, []);

    return compose(layerChain)(ctx, next);
  };

  dispatch.router = this;

  return dispatch;
};
```

`routes`根据`path`匹配`layer`。

同时，`router`还增加一个`allowedMethods`方法，主要是在没有匹配路由或者方法时进行处理，（在`routes`中如果匹配到路径，`koa`会在中间件方法处理处跳出，不会走到`allowedMethods`的方法。）

```
Router.prototype.allowedMethods = function (options) {
  options = options || {};
  var implemented = this.methods;

  return function allowedMethods(ctx, next) {
    return next().then(function() {
      var allowed = {};
      // 根据status 状态来判断
      if (!ctx.status || ctx.status === 404) {
        ctx.matched.forEach(function (route) {
          route.methods.forEach(function (method) {
            allowed[method] = method;
          });
        });

        var allowedArr = Object.keys(allowed);
        // 有两种没有匹配的情况：1，没有匹配的路径，2，有匹配的路径不支持对应的方法
        if (!~implemented.indexOf(ctx.method)) {
          if (options.throw) {
            var notImplementedThrowable;
            if (typeof options.notImplemented === 'function') {
              notImplementedThrowable = options.notImplemented(); // set whatever the user returns from their function
            } else {
              notImplementedThrowable = new HttpError.NotImplemented();
            }
            throw notImplementedThrowable;
          } else {
            ctx.status = 501;
            ctx.set('Allow', allowedArr.join(', '));
          }
        } else if (allowedArr.length) {
          if (ctx.method === 'OPTIONS') {
            ctx.status = 200;
            ctx.body = '';
            ctx.set('Allow', allowedArr.join(', '));
          } else if (!allowed[ctx.method]) {
            if (options.throw) {
              var notAllowedThrowable;
              if (typeof options.methodNotAllowed === 'function') {
                notAllowedThrowable = options.methodNotAllowed(); // set whatever the user returns from their function
              } else {
                notAllowedThrowable = new HttpError.MethodNotAllowed();
              }
              throw notAllowedThrowable;
            } else {
              ctx.status = 405;
              ctx.set('Allow', allowedArr.join(', '));
            }
          }
        }
      }
    });
  };
};
```

// router 匹配的实现
```
Router.prototype.match = function (path, method) {
  var layers = this.stack;
  var layer;
  var matched = {
    path: [],
    pathAndMethod: [],
    route: false
  };

  for (var len = layers.length, i = 0; i < len; i++) {
    layer = layers[i];

    debug('test %s %s', layer.path, layer.regexp);

    if (layer.match(path)) {
      matched.path.push(layer);

      if (layer.methods.length === 0 || ~layer.methods.indexOf(method)) {
        matched.pathAndMethod.push(layer);
        if (layer.methods.length) matched.route = true;
      }
    }
  }

  return matched;
};
```