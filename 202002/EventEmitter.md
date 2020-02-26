# EventEmitter

> 本质上就是发布订阅.方法有`on/off/emit/once`

```javaScript
function Event() {
    this.events = {}
}
Event.prototype.on = function(type, event) {
    if (this.events[type]) {
        this.events[type].push(event)
    } else {
        this.events[type] = [event]
    }
}
Event.prototype.emit = function(type) {
    this.events[type].map(evnet => {
        event()
    })
}
Event.prototype.off = function(type, event) {
    this.events[type] = this.events[type].filter(fn => {
        return fn !== event && fn !== event.cb
    })
}
Event.prototype.offAll = function(type, event) {
    this.events[type] = []
}
Event.prototype.once = function(type, event) {
    let that = this
    let wrap = function() {
        event()
        that.off(type, event)
    }
    wrap.cb = event
    this.off(type, wrap)
}
```
