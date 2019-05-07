# pm2

## 目录结构：

主要目录结构如下：

```bash
├── bin
│   ├── pm2    # 作为pm2命令的入口文件
│   ├── pm2-dev
│   ├── pm2-docker
│   └── pm2-runtime
├── constants.js    # 配置文件
├── index.js    # 作为npm包的入口文件
├── lib
│   ├── API
│   │   ├── CliUx.js
│   │   ├── Configuration.js
│   │   ├── Containerizer.js
│   │   ├── Dashboard.js
│   │   ├── Deploy.js
│   │   ├── Extra.js
│   │   ├── Log.js
│   │   ├── LogManagement.js
│   │   ├── Modules
│   │   │   ├── HTTP.js
│   │   │   ├── LOCAL.js
│   │   │   ├── Modularizer.js
│   │   │   ├── NPM.js
│   │   │   ├── TAR.js
│   │   │   ├── flagExt.js
│   │   │   ├── flagWatch.js
│   │   │   └── index.js
│   │   ├── Monit.js
│   │   ├── Serve.js
│   │   ├── Spinner.js
│   │   ├── Startup.js
│   │   ├── Version.js
│   │   ├── interpreter.json
│   │   ├── pm2-plus
│   │   │   ├── PM2IO.js
│   │   │   ├── auth-strategies
│   │   │   │   ├── CliAuth.js
│   │   │   │   └── WebAuth.js
│   │   │   ├── helpers.js
│   │   │   ├── link.js
│   │   │   ├── pres
│   │   │   │   ├── motd
│   │   │   │   ├── motd.update
│   │   │   │   └── welcome
│   │   │   └── process-selector.js
│   │   └── schema.json
│   ├── API.js    # 主程序目录
│   ├── Client.js
│   ├── Common.js
│   ├── Configuration.js
│   ├── Daemon.js
│   ├── Event.js
│   ├── God
│   │   ├── ActionMethods.js
│   │   ├── ClusterMode.js
│   │   ├── ForkMode.js
│   │   ├── Methods.js
│   │   └── Reload.js
│   ├── God.js
│   ├── HttpInterface.js
│   ├── ProcessContainer.js
│   ├── ProcessContainerFork.js
│   ├── ProcessUtils.js
│   ├── Satan.js
│   ├── TreeKill.js
│   ├── Utility.js
│   ├── Watcher.js
│   ├── Worker.js
│   ├── binaries
│   │   ├── DevCLI.js
│   │   ├── Runtime.js
│   │   └── Runtime4Docker.js
│   ├── completion.js
│   ├── completion.sh
│   ├── motd
│   ├── templates
│   │   ├── Dockerfiles
│   │   │   ├── Dockerfile-java.tpl
│   │   │   ├── Dockerfile-nodejs.tpl
│   │   │   └── Dockerfile-ruby.tpl
│   │   ├── ecosystem-simple.tpl
│   │   ├── ecosystem.tpl
│   │   ├── init-scripts
│   │   │   ├── launchd.tpl
│   │   │   ├── openrc.tpl
│   │   │   ├── pm2-init-amazon.sh
│   │   │   ├── rcd-openbsd.tpl
│   │   │   ├── rcd.tpl
│   │   │   ├── systemd-online.tpl
│   │   │   ├── systemd.tpl
│   │   │   └── upstart.tpl
│   │   └── logrotate.d
│   │       └── pm2
│   └── tools
│       ├── Config.js
│       ├── IsAbsolute.js
│       ├── find-package-json.js
│       ├── fmt.js
│       ├── isbinaryfile.js
│       ├── json5.js
│       ├── open.js
│       ├── passwd.js
│       ├── promise.min.js
│       └── xdg-open
├── package.json
├── paths.js    # pm2 默认配置目录
├── types
│   ├── index.d.ts
│   └── tsconfig.json
└── unit_time
```