# Buffer

> `Buffer`是`nodejs`中已经内置的处理二进制的模块。`Buffer`中文就是缓冲的意思，对，就是之前`stream`中的缓冲。一个很经常看到例子的就是视频等待缓冲后才能播放。

在`nodejs`中，`buffer`属于在`V8`堆外部内存创建的固定大小的内存。在使用`Buffer.from`、`Buffer.alloc`与`Buffer.allocUnsafe`传参为字符串、数组与`Buffer`时，则会进行一个深拷贝。如果给的是`ArrayBuffer`、`SharedArrayBuffer`则是同一个地址的引用。

`allocUnsafe`当创建内存小于`4kB`时，会从预分配的Buffer的切割出来。`allocUnsafeSlow`则是额外创建一个非内存池的`Buffer`拷贝。

运行`node  --zero-fill-buffers`时创建的`Buffer`直接用`0`填充。

### 方法

`Buffer`可以用`for...of`、`values`、`keys`与`entries`

```javascript
const arr = [1,2,3,4]
const bf = Buffer.from(arr)
console.log(bf.values())
```

---

Buffer.allocUnsafe 创建未初始化的固定大小的内存。速度比alloc快，但是不安全。因为可能有旧数据，解决办法：重写覆盖fill桌write