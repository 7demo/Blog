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

1. 验证`webpack`参数是否正确

```javaScript
const webpackOptionsValidationErrors = validateSchema(
    webpackOptionsSchema,
    options
);
if (webpackOptionsValidationErrors.length) {
    throw new WebpackOptionsValidationError(webpackOptionsValidationErrors);
}
```

2. 判断`webpack`的参数是对象或者是数组，分别对应了是单页面应用还是多页面应用。如果是多页面，则会遍历分别执行`webpack(option)`。

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

4. 获得`node`环境的能力

```javaScript
new NodeEnvironmentPlugin({
    infrastructureLogging: options.infrastructureLogging
}).apply(compiler);
```

5. 执行参数中的插件

```javaScript
if (options.plugins && Array.isArray(options.plugins)) {
    for (const plugin of options.plugins) {
        if (typeof plugin === "function") {
            plugin.call(compiler, compiler);
        } else {
            plugin.apply(compiler);
        }
    }
}
```

6. 执行`environment`与`afterEnvironment`的钩子

```javaScript
compiler.hooks.environment.call();
compiler.hooks.afterEnvironment.call();
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


