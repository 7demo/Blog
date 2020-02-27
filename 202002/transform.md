# 隐式转换

### 规则

1，算法运算符`+`会把其他数据类型变成number类型再进行; 2，字符串拼接符`+`会把其他数据类型变成`String`。怎么界定是算术运算符还是字符串拼接符号：

    1，`+`号两边只有要一个是字符串就是字符串拼接符

    2, `+`两边都是数字就是算符符号


2，关系运算符会把其他数据转数字再比较。

    1，两边如果一个是数字一个非数字，则非数字会转成数字再比较

    2，两边都是字符串，则会转成对应的unicode编码进行比较，比较时是从第一个字符依次比较下去

    3, NaN与谁都不相等


3，复制数据类型会先转成string，再转number，再运算。

    1，会先调用valueOf获取原始值，再用toString转字符串，再使用Number。

4, 空数组使用toString会得到空字符串。Boolean在`0, -0, NaN, undefinde, null, '', false, document.all()`外是true。

5, `{}与[]`不能对比。两边都是`[]`或者`{}`时，则是引用地址的比较，必定为false

### 题目

##### 1.

```javaScript
console.log(1+'true') //1true

console.log(1 + true) // 1 + 1  = 2

console.log(1 + undefined) // 1 + Nan = NaN

console.log(1 + null) // 1 + 0 = 1 , false 也表示0
```

##### 2.

```javaScript

"2" > 10 // false

"2" > "10" // true

'120' > '1150' // true

'abc' > 'b' // false

'abc' > 'abd' // true

```


##### 3，

```javaScript
[1,2] == '1,2'  // [1,2].valueOf().toString()
```

比如现在有一个变量`a`,如何当a等于多个值。

```javaScript

var a = {
    value: 1,
    valueOf: () => {
        let tmp = a.value
        a.value++
        return tmp
    }
}
if (a == 1 && a == 2 && a == 3) {
    ret【urn true
}

```


##### 4

```javaScript

[] == 0 // true

![] == 0 // true. ！！[]是得到一个true 然后取反false ==0

[] == [] // false  引用地址

[] == ![] // true 这个地方是空字符与逻辑非进行比较

{} == {} // false 引用地址

{} == !{} // false [object Obeject] 与false的对比

```
