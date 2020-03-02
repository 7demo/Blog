# 柯里化 与 尾递归

### 柯里化

> 高阶函数：接受多个参数的函数改变成接受一个单一参数的函数，返回的函数接受余下参数还能返回结果。有点可疑参数复用、参数确认、延迟执行

```javaScript

// 平常形式
function add (a, b, c) {
    return a + b + c
}

// 柯里化形式
function add(a) {
    return function(b){
        return function(c) {
            return a + b + c
        }
    }
}

```

通用版本柯里化：

```javaScript
function curry(fn, ...arg) {
    return function(..._arg) {
        return fn.call(null, ...arg, ..._arg)
    }
}
```

### 尾递归

拿斐波那契数列来距离：

```javaScript
function Fi(x) {
    if (x == 1 || x == 2) {
        return 1
    }
    return Fi(x - 1) + Fi(x - 2)
}
```

这个方法采用了递归，当数字过大的时候，必然会发生栈溢出的现象。我们可以改成遍历：

```javaScript
function Fi(x) {
    let num1 = 1
    let num2 = 1
    let sum = num1
    for (let i = 3; i <= x; i++) {
        sum = num1 + num2
        num1 = num2
        num2 = sum
    }
    return sum
}
```

但是递归，我们可以采用尾调用的方式——正常情况下，函数调用会形成函数栈，内存溢出也是因为调用栈过多（重复计算数值），这里尾调用就是只执行最后一步调用，不保存调用栈（不再借用外部环境变量）。每次把此次结果作为参数传递。

```javaScript
function Fi(n, x, y) {
    if (n == 1) {
        return x
    }
    return Fi(n - 1, y, x + y)
}
```

不过js中尾递归还是回发生爆栈，如不能不存在变量控制递归的情况

```javaScript
function sum(x, y) {
    if (y > 0) {
        return sum(x + 1, y - 1)
    } else {
        return x
    }
}
```

需要使用蹦床函数来优化：

```javaScript
function trampoline(f) {
    while (f && f instanceof Function) {
        f = f()
    }
    return f
}
```

上述函数改下，要返回一个函数：

```javaScript
function sum1(x, y) {
    if (y > 0) {
        return sum.bind(null, x + 1, y - 1)
    } else {
        return x
    }
}

trampoline(sum1(1, 1000))
```

但是蹦床函数没有做到真正的优化，以下才是：

```javaScript
function fco(f) {
    let value
    let active = false
    // 存放参数
    let accumulated = []
    return function acc() {
        console.log(arguments, accumulated)
        accumulated.push(arguments)
        if (!active) {
            active = true
            while (accumulated.length) {
                value = f.apply(this, accumulated.shift())
            }
            active = false
            return value
        }
    }
}
let sum = fco(function(x, y) {
    if (y > 0) {
        return sum(x + 1, y - 1)
    } else {
        return x
    }
})
```
