# libuv

> `nodejs`是异步编程，主要是基于`c`的`libuv`来实现的，所以这里有必要粗粗的学习下。这对异步编程的书写、`node`的`event-loop`将会更加了解。

## Reactor 模式

最初开始编写代码时，都是通过不停的循环去监测`socket`，如有`socket`连接进来，则去进行处理，伪代码如下：

```javascript
while(true) {
    socket = accept()
    handle(socket)
}
```

它的缺点一目了然，如果`handle`耗时比较久，则会阻塞新的`socket`连接。服务器处理能力极差。

基于以上，自然而言的衍生了多进程的处理方式：

```javascript
run() {
    socket = new Socket(port)
    new Thread(new Handler(socket.accept())).start()
}

function Handler(socket) {
    hander(socket)
}
```

监听接口进来的每个`socket`，都创建一个线程去处理。它缺点是创建线程与不停的创建、销毁线程也比较消耗资源。

在这儿就引出了`Reactor`模式。

> 此处handler直接是在线程池中执行的。

```javascript
socket = Socket.bind(port)
// 不停的轮询注册的操作
while (selecor.selectSocket()) {
    // 如果是接收就绪，则可以创建连接
    if (selectScoket.isAcceptAble) {
        // 注册socket
        soocket.register(selector)
    // 如果选择器是读就绪状态（处理后）
    } else (selectScoket.isReadable) {
        // read buffer 读取操作
        socket.close()
    }
}
```

其实，它抽象出来两个组件：

+ `Reactor`。负责响应IO，当监测到有新事件则给handler去处理，此时会创建`channel`。

+ `Handler`。则读取`channel`，处理业务逻辑后，再把结果写入`channel`。

## libuv

> libuv它提供一个`event-loop`，基于`io`和其他时间通知的回调函数。还提供其他工具：定时器，非阻塞网络支持，异步文件系统访问，子进程。


