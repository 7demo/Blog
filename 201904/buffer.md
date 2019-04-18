# Buffer

> `Buffer`是`nodejs`中已经内置的处理二进制的模块。`Buffer`中文就是缓冲的意思，对，就是之前`stream`中的缓冲。一个很经常看到例子的就是视频等待缓冲后才能播放。

在`nodejs`中，`buffer`属于在`V8`堆外部内存创建的固定大小的内存。`Buffer`的实例也是`Uint8Array`的实例。

在使用`Buffer.from`、`Buffer.alloc`与`Buffer.allocUnsafe`传参为字符串、数组与`Buffer`时，则会进行一个深拷贝。如果给的是`ArrayBuffer`、`SharedArrayBuffer`则是同一个地址的引用。

`allocUnsafe`当创建内存小于`4kB`时，会从预分配的Buffer的切割出来。`allocUnsafeSlow`则是额外创建一个非内存池的`Buffer`拷贝。

> Buffer.allocUnsafe 创建未初始化的固定大小的内存。速度比alloc快，但是不安全。因为可能有旧数据，解决办法：重写覆盖fill

运行`node  --zero-fill-buffers`时创建的`Buffer`直接用`0`填充。

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