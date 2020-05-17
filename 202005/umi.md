# umi脚手架探究

根据脚手架中`package.json`中的配置

```json
"bin": {
    "umi": "bin/umi.js"
}
```

定位到cli的入口文件为`bin/umi.js`

```javaScript
// 指定node 运行
#!/usr/bin/env node

// resolve-cwd是解析路径，不过是以process.cwd()为基准
const resolveCwd = require('resolve-cwd');

const { name, bin } = require('../package.json');
const localCLI = resolveCwd.silent(`${name}/${bin['umi']}`);
// 根据路径有个判断，进入不同的文件夹，此处从lib/cli开始
if (!process.env.USE_GLOBAL_UMI && localCLI && localCLI !== __filename) {
  const debug = require('@umijs/utils').createDebug('umi:cli');
  debug('Using local install of umi');
  require(localCLI);
} else {
  require('../lib/cli');
}
```

`lib/cli.js`核心代码

```javaScript
_asyncToGenerator(function* () {
  try {
    switch (args._[0]) {
      //  这个根据启动参数进行判断 也是yarn start --- umi dev
      // 也是调用fork 方法，参数为forkedDev
      case 'dev':
        // 用逗号是为了this指向global
        const child = (0, _fork.default)({
          scriptPath: require.resolve('./forkedDev')
        }); // ref:
        // http://nodejs.cn/api/process/signal_events.html

        process.on('SIGINT', () => {
          child.kill('SIGINT');
        });
        process.on('SIGTERM', () => {
          child.kill('SIGTERM');
        });
        break;

      default:
        const name = args._[0];

        if (name === 'build') {
          process.env.NODE_ENV = 'production';
        }

        yield new _ServiceWithBuiltIn.Service({
          cwd: (0, _getCwd.default)(),
          pkg: (0, _getPkg.default)(process.cwd())
        }).run({
          name,
          args
        });
        break;
    }
  } catch (e) {
    console.error(_utils().chalk.red(e.message));
    console.error(e.stack);
    process.exit(1);
  }
})();
```

`forkedDev`：

```javaScript
_asyncToGenerator(function* () {
  try {
    // 指定环境变量
    process.env.NODE_ENV = 'development';

    const service = new _ServiceWithBuiltIn.Service({
      cwd: (0, _getCwd.default)(),
      pkg: (0, _getPkg.default)(process.cwd())
    });
    yield service.run({
      name: 'dev',
      args
    });
    let closed = false; // kill(2) Ctrl-C

    process.once('SIGINT', () => onSignal('SIGINT')); // kill(3) Ctrl-\

    process.once('SIGQUIT', () => onSignal('SIGQUIT')); // kill(15) default

    process.once('SIGTERM', () => onSignal('SIGTERM'));

    function onSignal(signal) {
      if (closed) return;
      closed = true; // 退出时触发插件中的onExit事件

      service.applyPlugins({
        key: 'onExit',
        type: service.ApplyPluginsType.event,
        args: {
          signal
        }
      });
      process.exit(0);
    }
  } catch (e) {
    console.error(_utils().chalk.red(e.message));
    console.error(e.stack);
    process.exit(1);
  }
})();
```

`_ServiceWithBuiltIn`:

```javaScript
class Service extends _core().Service {
  constructor(opts) {
    process.env.UMI_VERSION = require('../package').version;
    process.env.UMI_DIR = (0, _path().dirname)(require.resolve('../package'));
    super(_objectSpread({}, opts, {
      presets: [require.resolve('@umijs/preset-built-in'), ...(opts.presets || [])],
      plugins: [require.resolve('./plugins/umiAlias'), ...(opts.plugins || [])]
    }));
  }

}
exports.Service = Service;
```

`_core`

```javaScript

```
