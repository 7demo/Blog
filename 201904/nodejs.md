# nodejs实战笔记

---

### 第一章

`node`包含新的es特性可以分为三类——`shipping`、`staged`与`in process`。其中`shipping`的特性是默认开启的。`staged`与`in process`需要命令开启——`harmony`。我们还可以使用以下命令查看各个阶段的特性：

`node --v8-options | grep "in porcess"`
