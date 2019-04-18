# ArrayBuffer

> `ArrayBuffer`、`TypeArray`、`DataView`是`js`操作二进制数据的一个接口，最初是满足与显卡间的数据交换，此规范在`es6`纳入规范。浏览器场景下，`File API`、`XMLHttpRequest`、`Fetch API`、`Canvas`、`WebSocket`。

## 大端字节序、小端字节序

字节序，超过1个字节的数据在内存中的存储顺序。

高低位，是指左边为高位，右边是低位。比如十进制的`123456789`，左边是高位、右边是低位。

大端字节序是最高有效字节（最大权重值，可以理解为首位）在高位，小端字节序是最低有效字节（最小权重值，可以理解为最低位）在高位。比如16进制的`0x12345678`，在内存中的顺序如果为大字节序则为`12345678`，如果为小端字节序为`87654321`。

区分的原因是，计算机先处理低位，所以计算机内都是小端字节序，为了高效率。而传输过程中，则大都是大端字节序。

## ArrayBuffer

代表内存中（栈中，比堆中的数组更快）的二进制数据，可以通过“视图”部署数组的接口进行操作。它不能直接读写数据，必须通过`TypedArray`、`DataView`来生成读写。

视图就是解读数据的方式。

## TypedArray

用于生成内存的视图——确定格式的二进制数据。

它与`DataView`区别是，`TypedArray`是一组构造函数，分别代表不同的数据格式。比如：`Int32Array`/`Uint8Array`等。而`dataView`是通过不同属性来操作的。

`TypedArray`可以接受三个参数，分别为`ArrayBuffer`、开始字节位置、结束字节位置。

`TypedArray`也可以接受参数为一个数字，直接初始化内存。

`TypedArray`可以以另外一个`TypedArray`作为参数，开辟新内存后复制了值。

`TypedArray`可以以`ArraylikeObj`作为参数，开辟新内存后复制了值。

`TypedArray.buffer`是获取`ArrayBuffer`对象。`TypedArray.byteLength`返回的是字节长度，而`TypeArray.length`返回的成员长度。

```javascript
var buffer = new ArrayBuffer(8)
var vb = new Uint16Array(buffer)
console.log(vb.length, vb.byteLength) // 4, 8 由于Uint16Array每个成员占用字节数是2，所以成员长度为4.
```

## DataView

也用于生成内存的视图——不确定格式的二进制数据（可以区分大小字端序）。比如：`Unit8`、`Init16`、`Float32`。


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

### 字符串与`arraybuffer`互相转换

字符串转`arraybuffer`:

```javascript
function str2ab(str) {
	// 一个字符串占用两个字节
	var buffer = new ArrayBuffer(str.length * 2)
	var bv = new Uint16Array(buffer)
	for (let i = 0; i < str.length; i++) {
		bv[i] = str.charCodeAt(i)
	}
	return buffer
}
```

`arraybuffer`转字符串

```javascript
function ab2str(buffer) {
	return String.fromCharCode.apply(null, new Uint16Array(buffer)) // 必须知道编码格式，js内部为utf-16
}
```

### 应用

在`XMLHttpRequest`第二版本中，支持了二进制数据的返回。首先在服务端创建一个路由，返回`buffer`。

```javascript
router.post('/buffer', (ctx, next) => {
	let buffer = Buffer.from('abcd')
	ctx.body = buffer
})
```

在页面上，我们使用`axios`：

```javascript
axios({
	url: '/buffer',
	method: 'POST',
	responseType: 'arraybuffer'
}).then((res) => {
	console.log(res.data)
})
```

可以看到，输出为：

```javascript
ArrayBuffer(4) {}
[[Int8Array]]: Int8Array(4) [97, 98, 99, 100]
[[Int16Array]]: Int16Array(2) [25185, 25699]
[[Int32Array]]: Int32Array [1684234849]
[[Uint8Array]]: Uint8Array(4) [97, 98, 99, 100]
byteLength: 4
__proto__: ArrayBuffer
```

那根据之前把`arraybuffer`转字符串的方法，我们可以这样写（`Buffer`的实例是`Uint8Array`的实例）：

```javascript
function ab2str(buffer) {
	return String.fromCharCode.apply(null, new Uint8Array(buffer))
}
ab2str(res.data) // abcd
```

此外，常见的应用场景：

· 当`websocket`设置`ws.binaryType=arrayBuffer`，也是通过二进制进行传输。

· 文件的读取——`FILE API`。

```
let file = document.querySelector('#input')
let reader = new FileReader()
reader.readAsArrayBuffer(file.files[0])
reader.onload = () => {
	console.log(reader.result)
}
```

· 浏览器中，主窗口与不同`work`通信时可以使用`SharedArrayBuffer`。与`ArrayBuffer`区别在于可共享。