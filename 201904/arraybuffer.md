# ArrayBuffer

> `ArrayBuffer`、`TypeArray`、`DataView`是`js`操作二进制数据的一个接口，最初是满足与显卡间的数据交换，此规范在`es6`纳入规范。浏览器场景下，`File API`、`XMLHttpRequest`、`Fetch API`、`Canvas`、`WebSocket`。

## 大端字节序、小端字节序

字节序，超过1个字节的数据在内存中的存储顺序。

高低位，是指左边为高位，右边是低位。比如十进制的`123456789`，左边是高位、右边是低位。

大端字节序是最高有效字节（最大权重值，可以理解为首位）在高位，小端字节序是最低有效字节（最小权重值，可以理解为最低位）在高位。比如16进制的`0x12345678`，在内存中的顺序如果为大字节序则为`12345678`，如果为小端字节序为`87654321`。

区分的原因是，计算机先处理低位，所以计算机内都是小端字节序，为了高效率。而传输过程中，则大都是大端字节序。

## ArrayBuffer

代表内存中的二进制数据，可以通过“视图”部署数组的接口进行操作。它不能直接读写数据，必须通过`TypedArray`、`DataView`来生成读写。

视图就是解读数据的方式。

## TypedArray

用于生成内存的视图——确定格式的二进制数据。

它与`DataView`区别是，`TypedArray`是一组构造函数，分别代表不同的数据格式。比如：`Int32Array`/`Uint8Array`等。而`dataView`是通过不同属性来操作的。

`TypedArray`可以接受三个参数，分别为`ArrayBuffer`、开始字节位置、结束字节位置。

## DataView

也用于生成内存的视图——不确定格式的二进制数据。比如：`Unit8`、`Init16`、`Float32`。


```javascript
var buf = new ArrayBuffer(32)
```

代表生成一个32字节大小的内存区域。如果我们继续对这块内存区域进行读取：

```javascript
# TypedArray
var buf = new ArrayBuffer(32)
var x1 = new Int32Array(buf)
x1[0] = 1
console.log(x1, x1[0])
// Int32Array [ 1, 0, 0, 0, 0, 0, 0, 0 ] 1
```

需要知道的是，如果基于同一内存数据建立多个视图进行操作，则数据会相互影响:

```javascript
var buf = new ArrayBuffer(32)
var x1 = new Int32Array(buf)
x1[0] = 1
var x2 = new Int8Array(buf)
x2[0] = 2
console.log(x1, x1[0])
// Int32Array [ 2, 0, 0, 0, 0, 0, 0, 0 ] 2
```

`TypedArray`不但可以用`ArrayBuffer`作为参数，还可以用数组作为参数，直接初始化内存后赋值。

由于内存有可能不足的情况，初始化可能失败。所以，我们初始化后可以使用`buf.byteLength === n`来判断是否成功。

`buf.slice(0, n)`可以拷贝原对象。

`ArrayBuffer.isView(buf)`来判断是否为一个视图的实例。
