# 前端安全

### CSRF

> Cross-site requset forgery，跨站请求伪造。第三方网站在风险网站用户没有退出的情况下，发起一个请求，带有cookie。

防御主要是两部分，一个是服务端，一个是客户端，主要是服务端：

1，每次请求带有token

2，只接受post请求

3，监测referer

### XSS

> cross-site script，跨站脚本攻击。网站输入恶意的html代码，执行。可以盗用cookie、重定向到其他网站、破坏页面结构。

防御：

1，过滤用户输入

2，保护好cookie，httponlue=true

### SQL注入

> sql注入插入表单提交

防御：

1，不相信用户的输入

2，不动态拼接sql，使用参数化的SQL

3，避免使用管理员权限连接数据库

4，敏感信息避免直接存放

### DDOS共计

> 大量合法请求占用大量资源，以谈话网络。

### SYN攻击

> 等待三次握手中的第三次ack

防护：

1，系统存在一个`backlog`（你默认1024字节）队列，来维护tcp连接与SYN-RECEIVED状态。所以增加控制`backlog`的大小。

2，缩短超时时间即syn-received时间

3，过滤网关与防火墙

4，限制单ip的链接数目，这针对没有伪装的攻击。同时针对伪装的攻击，可以维护恶意ip列表甚至子网段。


### 网络劫持攻击

防御：HTTPS

### 控制台注入代码

### iframe安全

防御: 设置`sandbox`属性，最小权限控制（不能提交表单、弹窗、执行脚本）。但是也支持配置：

    1, `allow-forms`

    2, `allow-popups`

    3, `allow-scripts`

    4, `allow-same-origin`

### 点击劫持

> 视觉欺骗，透明iframe覆盖网页上。

防御：

1，设置相应中的X-frame-options：deny(不能嵌套任何iframe或者嵌套任何frame中)、sameorigin(只能嵌套或者被嵌套当前域)、allow-from uri(白名单)

### 错误内容推断

> 上传图片，却上传了js文件

防御：

1, 设置X-Content-Type-Options来禁止浏览器去推断响应，`nosniff`
