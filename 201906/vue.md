# Vue源码学习

> 参考源码自我实现一个`Tue`。参考`vue`版本为`2.6.8`。

## 目录结构

> 我们也参考`vue`，使用`rollup`进行构建我们的程序。

```
├── dist # 编译后的文件
│   └── tue.js
├── example # 例子文件
├── package.json
├── scripts # rollup配置文件
│   └── config.js
├── src # 源文件
│   └── index.js
```

## Tue

看`vue`的文档，可以知道`Tue`是构函数。

```javascript
function Tue(options) {
}
```

