# Buffer

> `Buffer`是`nodejs`中已经内置的处理二进制的模块。`Buffer`中文就是缓冲的意思，对，就是之前`stream`中的缓冲。一个很经常看到例子的就是视频等待缓冲后才能播放。

在`nodejs`中，`buffer`属于在`V8`堆外部内存创建的固定大小的内存。`Buffer`的实例也是`Uint8Array`的实例。可以通过下个例子来窥视下。

```javascript
var bf = Buffer.alloc(1)
console.log(bf.buffer) // ArrayBuffer { byteLength: 1 }
```

> 在下载文件时，服务端返回的数据，可以很容易找到`Unit8Array`对象。

```javascript
axios({
	url: '/buffer',
	method: 'POST',
	responseType: 'arraybuffer'
}).then((res) => {
	console.log(res.data)
})
```

在使用`Buffer.from`、`Buffer.alloc`与`Buffer.allocUnsafe`传参为字符串、数组与`Buffer`时，则会进行一个深拷贝。如果给的是`ArrayBuffer`、`SharedArrayBuffer`则是同一个地址的引用。

`allocUnsafe`当创建内存小于`4kB`时，会从预分配的Buffer的切割出来。`allocUnsafeSlow`则是额外创建一个非内存池的`Buffer`拷贝。

> Buffer.allocUnsafe 创建未初始化的固定大小的内存。速度比alloc快，但是不安全。因为可能有旧数据，解决办法：重写覆盖fill

> 运行`node  --zero-fill-buffers`时创建的`Buffer`直接用`0`填充。

### 创建

`Buffer.from(ArrayBuffer)`返回`buffer`（需`arraybuffer`的`buffer`属性），与给定的`arrayBuffer`共享内存。

```javascript
const bf = new Uint16Array(2)
bf[0] = 1
bf[1] = 2
const buffer = Buffer.from(bf.buffer)
buffer[0] = 3
bf[0] === 3 // true
```

`Buffer.from(string | array | buffer)`是给定对象的拷贝。

### 方法

`Buffer`可以用`for...of`、`values`、`keys`与`entries`

```javascript
const arr = [1,2,3,4]
const bf = Buffer.from(arr)
console.log(bf.values())
```

此外，还有比较`Buffer.compare`、合并`Buffer.concat`。

当我们需要改变`buffer`的长度，最好通过`slice`来指向统一内存新的`buffer`。`slice`接受两个参数，开始位置与结束位置，当值为负的时候，则是相当于`buffer`的末尾。

```javascript
buffer.slice(-6, -1) // 相当于buffer.slice(0, 5)
```

`buf.readDoubleBE`返回大端序，`buf.readDoubleLE`返回小端序。参数为偏移量。

`buf.swap16()`交互字节顺序，不过字节长度得为2的倍数。

```javascript
const buffer = Buffer.from([1,2,3,4])
buffer.swap16()
// <Buffer 01 02 03 04>
console.log(buffer)
// <Buffer 02 01 04 03>
```

`buffer.constants.MAX_LENGTH`返回单个内存实例允许的最大内存

`buffer.constants.MAX_STRING_LENGTH`返回单个内存示例允许最大长度

---

### 应用

#### `buffer`的拼接

`buffer`提供了一个方法是`buffer.concat`，专门用于合并`buffer`。接受两个参数，第一个参数为`buffer`的数组，第二个参数是合并后`buffer`的长度，如果长度没有设置，则会默认去数组中每个`buffer`的长度和。

```javascript
const bf1 = Buffer.alloc(2)
const bf2 = Buffer.alloc(3)
const bf3 = Buffer.alloc(4)
bf1[0] = 1
bf1[1] = 1
bf2[0] = 2
bf2[1] = 2
bf2[2] = 2
bf3[0] = 3
bf3[1] = 4
bf3[2] = 4
bf3[3] = 4
const bf4 = Buffer.concat([bf1, bf2, bf3])
console.log(bf4.length, bf4) // 9 <Buffer 01 01 02 02 02 03 04 04 04>

const bf1 = Buffer.from('111')
const bf2 = Buffer.from('2222')
const bf3 = Buffer.from('33333')
const bf4 = Buffer.concat([bf1, bf2, bf3])
console.log(bf4.length, bf4.toString()) // 12 '111222233333'

```

#### `buffer`存储与传输

在常用的http请求中，其实都是`buffer`在传输，由`buffer`组成`stream`。所以理论上，我们把字符串或者json对象转成`buffer`会更快。但是由于字符串的处理远比`buffer`快，所以该用字符串的地方还是需要使用字符串。

```javascript
var time = 300*1000;
var txt = "aaa"

var str = "";
console.time('test5')
for(var i=0;i<time;i++){
    str += txt;
}
console.timeEnd('test5') // test5: 25.329ms

var time = 300*1000;
var txt = "aaa"
var bf = Buffer.from(txt)
console.time('testb')
for(var i=0;i<time;i++){
    bf = Buffer.concat([bf, Buffer.from(txt)], bf.length + 3)
}
console.timeEnd('testb') // testb: 53432.812ms

var time = 300*1000;
var txt = "aaa"
var bf = Buffer.alloc(time * txt.length)
var offset = 0
console.time('testb')
for(var i=0;i<time;i++){
    bf.write(txt, offset)
    offset += txt.length
}
console.timeEnd('testb') // testb: 40.887ms
```

由于`buffer`是在堆外内存中，也适合大对象的存储。


#### `buffer`的大转小


一般来说`gbk模式`下，一个字符占据两个字节，每个字节是8位；`utf-8`模式下，中文是3个字节，数字与英文是一个字节。js中一个一般是unicode编码模式。utf8下，时间戳`1556033783480`的字节长度为13，如果我们用`buffer`来保存呢。

这得知道`Buffer`的实例就是`Uint8Array`的实例。

```javascript
var s = 1556033783480
var bf = Buffer.alloc(6)
bf.writeUIntBE(s, 0, 6)
console.log(bf.length, bf.byteLength) // 6，6
console.log(bf.readUIntBE(0, 6)) // 1556033783480
```

这里`Buffer.writeUIntBE`是`buffer`的写入方法。接受三个参数，分别是值、开始地址与偏移量。其中偏移量是大于0小于6(单位字节)。由于每个字节是`8位`，那么最大它最大考验表示`2^(6*8) - 1`的值，即`281474976710655`。

我们把值设置成`281474976710655`看看：

```javascript
var s = 281474976710655
var bf = Buffer.alloc(6)
bf.writeUIntBE(s, 0, 6)
console.log(bf.length, bf.byteLength) // 6，6
console.log(bf.readUIntBE(0, 6)) // 281474976710655
```

如果我们把值设置成`281474976710657`

```javascript
var s = 281474976710657
var bf = Buffer.alloc(6)
bf.writeUIntBE(s, 0, 6)
console.log(bf.length, bf.byteLength) // 6，6
console.log(bf.readUIntBE(0, 6)) // 1
```

由于这个值已经越界，最后读出的值为`1`。

基于以上的理解，我们在存储一些数值的时候，可以使用`buffer`来存储。


#### 转码

`Buffer`与js字符串间需要显式的调用编码方法，包含以下几种：

· `ascii`

· `utf-8`

· `ucs2`

· `base64`

· `binary`（已废弃）

```javascript
var bf = Buffer.from('abc', 'utf-8')
console.log(bf.toString()) // abc
console.log(bf.toString('utf-8')) // abc
console.log(bf.toString('base64')) // YWJj
console.log(bf.toString('hex')) // 616263
```

我们可以使用`iconv-lite`来识别编码格式，甚至`gbk`转`utf-8`——识别编码格式，应该是根据`buffer`的索引为0、1的值来判断。

#### 乱码

乱码究其原因是汉字被不完整读取。比如汉字`utf-8`下占用3字节，如果我们从1或者2的位置读取就会出现。

```javascript
var bf = Buffer.from('你好，今天')
console.log(bf.byteLength, bf.toString()) // 15 你好，今天
var bf1 = bf.slice(0, 5)
console.log(bf1.byteLength, bf1.toString()) // 5 '你�'
```

在这种情况下，我们可以使用`string_decoder`模块来解码。

```javascript
var {StringDecoder} = require('string_decoder')
const decoder = new StringDecoder('utf8')
var bf = Buffer.from('你好，今天')
console.log(bf.byteLength, bf.toString()) // 15 你好，今天
var bf1 = bf.slice(0, 5)
console.log(decoder.write(bf1)) // 5 '你'
```

#### 文件上传

