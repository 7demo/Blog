# PWA

> 只能运营在localhost有HTTPS。

### 特点：

1，生成桌面图标

2，离线缓存

3，消息推送


### 使用

声明周期：`parsed/installing/installed/activing/actived/redundant`

> 注册服务


```javaScript
navigator.serviceWorker.register('sw.js').then(() => {

})
```
> sw.js

```javaScript
// 缓存文件
let cacheName = 'test'
let fileCache = [
    '/index.html'
    //...
]
self.addEventListener('install', function(e) {
    e.waitUntil(
        caches.open(cacheName).then(function(cache) {
            return cache.addAll(fileCache)
        })
    )
})

self.addEventLisrener('fetch', function(e) {
    e.responseWith(
        caches.match(e.request).then((response) => {
            // reponse使用时只能使用一次 所以是clone()
            return response
        })
    )
})
```

`Push`:服务器将消息推送至sw

`notification`: sw将消息推送至用户

### 图片配置

```javaScript
// manifest.json
{
    // icons
    // bg
    // theme-color
    // display
    // start-url
}
```
