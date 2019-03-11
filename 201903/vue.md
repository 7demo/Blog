# Vue源码研究学习

> 此源码学习以2.6.8版本为对象。

---

先构建`Vue`对象

```javascript

function Vue (options) {
  if (!(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword');
  }
  this._init(options);
}

```

> 如果是