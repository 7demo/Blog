# performance

> 可监控用户白屏事件、首屏时间、总时间、用户操作点。

### timeOrigin

> 时间基点

### onresourcetimingbufferfull

> 代表回调函数，浏览器的资源时间性能缓冲满了触发。

### memory

> 内存相关

1. `totalJSHeapSize`总内存大小

2，`usedJSHeapSize`已使用。如果`usedJSHeapSize`大于`total`则是内存泄漏。

3，`JSHeapLimit`内存限制大小

### navigation

> 页面来源信息

`redirectCount`:默认为0，页面如果是重定向过来的话，经过几次。

`type`: 默认为0，代表打开方式。

    1, 0表示正常打开

    2，1代表reload

    3，2代表前进后退进入页面

    4，255非以上方式

### timing

> 一系列时间节点

1, `navigationStart`代表同个窗口上个页面卸载时间，如果第一次进入则与`fetchStart`相同

2，`redirectStart`第一个重定向开始的时间戳，如果没有或者重定向到不同源则为0.

3, `redirectEnd`最后一个重定向结束的时间戳，如果没有或者定向不同源则为0

4，`fetchStart`，在检查缓存前，浏览器准备抓取HTTP资源文档的时间

5，`domainLookupStart`，dns查询开始时间，如果使用缓存或者持久连接，则与`fetchStart`相同

6, `domainLookupEnd`， dns查询结束时间，如果使用缓存或者持久连接，则与`fetchStart`相同

7，`connectStart`, HTTP开始建立链接时间，如果持久连接，则与`fetchstart`一样。但是如果断开重连，则为新的开始时间

8, `conentEnd`， HTTP完成建立链接时间(http握手)，如果持久连接，则与`fetchstart`一样。但是如果断开重连，则为新的开始时间

9，`sourceConnectionStart`: https建立链接时间。如果不是HTTPS，则为0

10， `requestStart`，读取文档真实时间，包括从缓存。

11, `responseStart`, 响应开始时间（接受第一个字节），包括从缓存

12， `responseEnd`, 响应结束时间（最后一个字节），包括从缓存

13，`unloadStart`, 前一个页面unload时间戳，如果没有则为0

14，`unloadEnd`，前一个页面unload对应完成的回调完成的时间戳，如果没有则为0

15，`domloading`，开始渲染dom树时间

16，`domInteractive`, dom解析完成时间（但是没有下载资源）

17，`domContentLoadedEventStart`dom解析完成，开始加载网页资源的时间

18，`domContentLoadedEventEnd`dom解析完成，完成资源加载时间

19，`domComplete`完成dom解析，资源加载完成。此时document.readState是complete，开始出发readStateChange

20，`loadEventStart` load时间发送文档，也是load函数开始执行时间，如果没有回调函数则为0

21， `loadEventEnd` load函数执行结束时间，如果没有则为0

### getEntries 每个对象的HTTP请求

### 指标计算

重定向耗时：`redirectEnd - redriectStart`

DNS查询耗时：`domainLookupEnd - domainLoopupStart`

TCP连接耗时：`connectEnd - connectStart`

HTTP请求耗时：`responseEnd-requestStart`

dom解析耗时：`domCompete-domInteractive`

白屏时间：`responseStart-navigationStart`

domready时间：`domContentLoadedEventEnd-navigationStart`

load时间：`loadEventEnd-navigationStart`
