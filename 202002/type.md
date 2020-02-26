# 类型判断

> 主要是 typeof/instanceof/toString

### typeof

`typeof`主要是判断除`null`外的基础类型，数字、字符串、布尔、`undefined`、`Symbol`，还可以判断函数。

```javaScript
typeof 1 // number
typeof 'str' // string
typeof true // boolean
typeof undefined // undefined
typeof function(){} // function
typeof Symbol() // symbol
```

### instanceof

`instanceof`是通过原型链判断的，用来判断是不是一个对象的实例，它用于类型判断是不够准确的。

```javaScript
[1] instanceof Array // true

[1] instanceof Object // true
```

### toString

```javaScript
console.log(Object.prototype.toString.call({})) // [object Object]
console.log(Object.prototype.toString.call([])) // [object Array]
console.log(Object.prototype.toString.call(function(){})) // [object Function]
console.log(Object.prototype.toString.call(new Date())) // [object Date]
console.log(Object.prototype.toString.call(new Set())) // [object Set]
console.log(Object.prototype.toString.call(new Map())) // [object Map]
console.log(Object.prototype.toString.call(/\w/)) // [object Regexp]
```
