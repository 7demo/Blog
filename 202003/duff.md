# Duff`s Device

主要是为了解决循环造成的性能底下，可以一个循环中执行多次。

```javaScript
function duff(items) {
    // 大概率不是8的倍数，则先把零头给执行
    let i = items.length % 8
    while (i) {
        process(items[i--])
    }
    i = Math.floor(item.length/8)
    while (i) {
        process(items[i--])
        process(items[i--])
        process(items[i--])
        process(items[i--])
        process(items[i--])
        process(items[i--])
        process(items[i--])
        process(items[i--])
    }
}
```
