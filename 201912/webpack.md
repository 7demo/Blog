# webpack

分析`webpack`的源码，我们需要从`webpack-cli`分析起，毕竟我们是直接在`shell`中直接使用`webpack`的。为了更好的研究源码，首先创建个demo:

```
├── dist
├── index.html
├── node_modules
├── package.json
├── src
    ├── css
    │   └── index.css
    └── main.js
├── webpack.config.js
└── yarn.lock
```

```javaScript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
    entry: path.join(__dirname, "./src/main.js"),//模块入口
    output: {
        path: path.join(__dirname, './dist'),//输出的配置文件的目录
        filename: "bundle.js"//输出配置文件的js
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: path.join(__dirname, "./index.html"),
            filename: "index.html"
        })
    ],
    mode: 'development',
    devtool: 'source-map',
    // watch: true,
    // watchOptions: {
    //   poll: 1000, // 每秒询问多少次
    //   aggregateTimeout: 500,  //防抖 多少毫秒后再次触发
    //   ignored: /node_modules/ //忽略时时监听
    // },
    module: {
        rules: [
            {
                test: /\.css$/, use: ["style-loader", "css-loader"]
            }
            // {
            //     test: /\.less$/, use: ["style-loader", "css-loader", "less-loader"]
            // }, {
            //     test: /\.scss$/, use: ["style-loader", "css-loader", "sass-loader"]
            // }
        ]
    }
}

```
## webpack-cli/webpack

安装`webpack-cli`时，会把`webpack`也装好。`webpack-cli`的入口文件是`webpack-cli/bin/cli.js`，这文件的主要作用是引入`webpack`生成`compiler`, 根据是否需要`watch`来运行。

```javaScript
// webpack-cli/bin/cli.js
// ...
const webpack = require("webpack");
let compiler;
try {
    compiler = webpack(options);
} catch (err) {
    throw err;
}
//
if (argv.progress) {
    const ProgressPlugin = require("webpack").ProgressPlugin;
    new ProgressPlugin({
        profile: argv.profile
    }).apply(compiler);
}
// 默认回调
function compilerCallback(err, stats) {
    // ...
}
// 如果是watch模式
if (firstOptions.watch || options.watch) {
    const watchOptions =
        firstOptions.watchOptions || options.watchOptions || firstOptions.watch || options.watch || {};
    if (watchOptions.stdin) {
        process.stdin.on("end", function(_) {
            process.exit(); // eslint-disable-line
        });
        process.stdin.resume();
    }
    // 执行的是compiler.watch
    compiler.watch(watchOptions, compilerCallback);
    if (outputOptions.infoVerbosity !== "none") console.error("\nwebpack is watching the files…\n");
} else {
    // 如果不是watch，直接run
    compiler.run((err, stats) => {
        if (compiler.close) {
            compiler.close(err2 => {
                compilerCallback(err || err2, stats);
            });
        } else {
            compilerCallback(err, stats);
        }
    });
}
```

在`webpack-cli`中，`require('wabpack')`是引用的`webpack`，它的入口文件时`webpack/lib/webpack.js`。在`webpack.js`中主要做了以下事情：

#### 1. 验证`webpack`参数是否正确

```javaScript
const webpackOptionsValidationErrors = validateSchema(
    webpackOptionsSchema,
    options
);
if (webpackOptionsValidationErrors.length) {
    throw new WebpackOptionsValidationError(webpackOptionsValidationErrors);
}
```

`validateSchema`是通过预定义参数的`schema`， 来通过`ajv`这个第三方库来做判断的。

#### 2. 判断`webpack`的参数是对象或者是数组，分别对应了是单页面应用还是多页面应用。如果是多页面，则会遍历分别执行`webpack(option)`。

```javaScript
compiler = new MultiCompiler(
    Array.from(options).map(options => webpack(options))
);
```

3. 结合默认参数，处理参数，生成`compiler`实例。

```javaScript
options = new WebpackOptionsDefaulter().process(options);
compiler = new Compiler(options.context);
compiler.options = options;
```

4. 获得`node`环境的能力——读写与watcher的能力，同时这个在`beforeRun`挂载钩子。

```javaScript
new NodeEnvironmentPlugin({
    infrastructureLogging: options.infrastructureLogging
}).apply(compiler);

class NodeEnvironmentPlugin {
    constructor(options) {
        this.options = options || {};
    }

    apply(compiler) {
        // 使用node console
        compiler.infrastructureLogger = createConsoleLogger(
            Object.assign(
                {
                    level: "info",
                    debug: false,
                    console: nodeConsole
                },
                this.options.infrastructureLogging
            )
        );
        compiler.inputFileSystem = new CachedInputFileSystem(
            new NodeJsInputFileSystem(),
            60000
        );
        const inputFileSystem = compiler.inputFileSystem;
        compiler.outputFileSystem = new NodeOutputFileSystem();
        compiler.watchFileSystem = new NodeWatchFileSystem(
            compiler.inputFileSystem
        );
        // 挂载beforeRun钩子
        compiler.hooks.beforeRun.tap("NodeEnvironmentPlugin", compiler => {
            if (compiler.inputFileSystem === inputFileSystem) inputFileSystem.purge();
        });
    }
}
```

5. 应用参数中的插件

```javaScript
// 此处的插件只有 HtmlWebpackPlugin
if (options.plugins && Array.isArray(options.plugins)) {
    for (const plugin of options.plugins) {
        if (typeof plugin === "function") {
            plugin.call(compiler, compiler);
        } else {
            // 应用插件HtmlWebpackPlugin
            plugin.apply(compiler);
        }
    }
}
```

6. 执行`environment`与`afterEnvironment`的钩子

```javaScript
compiler.hooks.environment.call();
compiler.hooks.afterEnvironment.call();
// 此处挂载 SingleEntryPlugin 钩子到make上
compiler.options = new WebpackOptionsApply().process(options, compiler);
```

7. 如果`callback`，则直接执行`compiler.run`，否则只暴露`compiler`。

```javaScript
if (callback) {
    if (
        options.watch === true ||
        (Array.isArray(options) && options.some(o => o.watch))
    ) {
        const watchOptions = Array.isArray(options)
            ? options.map(o => o.watchOptions || {})
            : options.watchOptions || {};
        return compiler.watch(watchOptions, callback);
    }
    compiler.run(callback);
}
```

8. 把默认参数、node环境能力等挂载到`webpack`上。

```javaScript
webpack.WebpackOptionsDefaulter = WebpackOptionsDefaulter;
webpack.WebpackOptionsApply = WebpackOptionsApply;
webpack.Compiler = Compiler;
webpack.MultiCompiler = MultiCompiler;
webpack.NodeEnvironmentPlugin = NodeEnvironmentPlugin;
// @ts-ignore Global @this directive is not supported
webpack.validate = validateSchema.bind(this, webpackOptionsSchema);
webpack.validateSchema = validateSchema;
webpack.WebpackOptionsValidationError = WebpackOptionsValidationError;
```

## Compiler

`Compiler`是继承`tapable`，在`hooks`对象中定义各种了钩子管理器——`shouldEmit/beforeRun`...，用于在对应的时机调用。

```javaScript
class Compiler extends Tapable {
    constructor(context) {
        super();
        this.hooks = {
            // 钩子管理
        }
    }
}
```

当执行`compiler.run()`时，首先通过`this.hooks.beforeRun.callAsync`，来执行`beforeRun`的钩子，再执行`run`的钩子。然后执行`readRecords`, 它的回调为`compile`

```javaScript
run(callback) {
    const finalCallback = (err, stats) => {
        // 回调
    };
    this.hooks.beforeRun.callAsync(this, err => {
        if (err) return finalCallback(err);
        this.hooks.run.callAsync(this, err => {
            if (err) return finalCallback(err);
            this.readRecords(err => {
                if (err) return finalCallback(err);
                this.compile(onCompiled);
            });
        });
    });
}
```

```javaScript
// readRecords
readRecords(callback) {
    if (!this.recordsInputPath) {
        this.records = {};
        return callback();
    }
    this.inputFileSystem.stat(this.recordsInputPath, err => {
        // It doesn't exist
        // We can ignore this.
        if (err) return callback();
        this.inputFileSystem.readFile(this.recordsInputPath, (err, content) => {
            if (err) return callback(err);
            try {
                this.records = parseJson(content.toString("utf-8"));
            } catch (e) {
                e.message = "Cannot parse records: " + e.message;
                return callback(e);
            }
            return callback();
        });
    });
}
```

在`compile()`时，会先后执行`beforeCompile`/`compile.call`/`make.callAsync`/`compilation.finish`/`compilation.seal`/`afterCompile.callAsync/shouldEmit/this.emitAssets`。这与之前`beforeRun`时触发的函数构成了整个`webpack`执行流程。

```javaScript
// compile
compile(callback) {
    const params = this.newCompilationParams();
    this.hooks.beforeCompile.callAsync(params, err => {
        if (err) return callback(err);
        this.hooks.compile.call(params);
        const compilation = this.newCompilation(params);
        this.hooks.make.callAsync(compilation, err => {
            if (err) return callback(err);
            compilation.finish(err => {
                if (err) return callback(err);
                compilation.seal(err => {
                    if (err) return callback(err);
                    this.hooks.afterCompile.callAsync(compilation, err => {
                        if (err) return callback(err);
                        return callback(null, compilation);
                    });
                });
            });
        });
    });
}
```

看到这里，就会发现`webpack`就是基于`tapable`的发布订阅实现的一个插件管理系统，通过在整个流程不同阶段挂载触发对应的钩子来实现模块的打包、处理、输出。


#### beforeRun

`beforeRun`挂载的钩子是在获取`node`环境能力时的`NodeEnvironmentPlugin`，这个地方会执行`inputFileSystem.purge()`, 主要是本地缓存数据初始化。

```javaScript
compiler.hooks.beforeRun.tap("NodeEnvironmentPlugin", compiler => {
    if (compiler.inputFileSystem === inputFileSystem) inputFileSystem.purge();
});
```

```javaScript
// purge
purge(what) {
    // 本地缓存初始化
    this._statStorage.purge(what);
    // 读取文件夹缓存初始化
    this._readdirStorage.purge(what);
    // 读取文件
    this._readFileStorage.purge(what);
    this._readlinkStorage.purge(what);
    this._readJsonStorage.purge(what);
}
// inputFileSystem.purge
purge(what) {
    if (!what) {
        this.count = 0;
        clearInterval(this.interval);
        this.nextTick = null;
        this.data.clear();
        this.levels.forEach(level => {
            level.clear();
        });
    } else if (typeof what === "string") {
        for (let key of this.data.keys()) {
            if (key.startsWith(what)) this.data.delete(key);
        }
    } else {
        for (let i = what.length - 1; i >= 0; i--) {
            this.purge(what[i]);
        }
    }
}
```
#### run.callAsync

`run`挂载的钩子是`CachePlugin`。

```javaScript
compiler.hooks.run.tapAsync("CachePlugin", (compiler, callback) => {
    // 就demo来说，_lastCompilationFileDependencies为undefined
    if (!compiler._lastCompilationFileDependencies) {
        return callback();
    }
});
```

#### readRecords

就`demo`来说，`readRecords`是直接执行了回调`compile`

#### compile函数

`compile`函数中，首先通过`newCompilationParams()`返回`params`:

```javaScript
const params = {
    normalModuleFactory: this.createNormalModuleFactory(),
    contextModuleFactory: this.createContextModuleFactory(),
    compilationDependencies: new Set()
};
```

`normalModuleFactory`与`contextModuleFactory`都是基于`tapable`的`module`工厂。

#### newCompilation

根据`this.newCompilation`生成`compilation`。

```javaScript
newCompilation(params) {
    const compilation = this.createCompilation();
    compilation.fileTimestamps = this.fileTimestamps;
    compilation.contextTimestamps = this.contextTimestamps;
    compilation.name = this.name;
    compilation.records = this.records;
    compilation.compilationDependencies = params.compilationDependencies;
    this.hooks.thisCompilation.call(compilation, params);
    this.hooks.compilation.call(compilation, params);
    return compilation;
}
```

`newcompilation`也是继承`tapable`。`compile`可以理解为编译器，控制着整个流程。而`compilation`则是一个编译对象，拥有整个过程产生的一切对象。


#### make.callAsync

`make`则是开始进行编译，会先后触发`HtmlWebpackPlugin`与`SingleEntryPlugin`钩子。

`SingleEntryPlugin`钩子会触发`compilation.addEntry`，`addEntry`主要是调用了`this._addModuleChain`


## webpack性能优化

## 插件机制

## 插件编写

## webpack打包模块
