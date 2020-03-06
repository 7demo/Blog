# 设计模式

### 单例模式

> 保证一个类只有一个实例

```javaScript
function A() {
    let instance = null
    function M() {
        this.a = 1
        this.setM = function(m) {
            this.a =m
        }
        this.getM = function() {
            return this.a
        }
    }
    return {
        getInstance:() => {
            if (!instance) {
                instance = new M()
            }
            return instance
        }
    }
}
let M = A()
var a = M.getInstance()
var b = M.getInstance()
```

### 代理模式

> 防抖函数与节流函数

### 发布订阅模式与观察者模式

> 观观察者模式由具体目标调度，而发布订阅模式由调度中心调度的。观察者模式中，发布者与订阅者存在依赖，而发布订阅模式中不存在依赖。

观察者函数比如`Vue`的实现，发布订阅模式比如`Eventemit`

### 工厂模式

### 缓存代理

> 第一次执行时候缓存，后来直接取缓存

### 策略模式
