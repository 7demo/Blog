# http权威指南

## 一 web基础

+ 通过设置`MIME(Multipurpose Internet Mail Extension)`（多用途因特网邮件扩展）设置相应类型，对应responsehead中的`content-type`。

+ URI 表示统一资源标识符，分为URL与URN。URL表示统一资源定位符，标识资源在网上的位置；URN表示统一资源名（未被广泛使用）。

+ get与post请求区别：

> 1.get请求数据在URL，post请求数据放到在请求体内

> 2.get的url最大长度为2048字符，即2个字节。post无限制。不过服务器一般限制64kb。

> 3.post除了ascii编码外，还支持2进制。

> 4，get回退刷新是无害，post会再次提交。并且get可以被缓存。

> 5. get是幂等，post是非幂等。

> 6. get与post的一个区别：post会把header与post分为两个tcp包，而get则是一个包上传的。不过只是某些浏览器实现的区别，比如firefox只发一次。

但是从tcp角度，二者是一样的。二者只是tcp的表现形式。

+ tcp提供"无差错传输、按序传输、未分段的数据流"

+ 网关：通常把http协议转为其他协议。

+ 隧道：在两条链接之前进行数据的盲目转发。http隧道通常在一条或者多条http链接上转发非http链接，比如ssl.
