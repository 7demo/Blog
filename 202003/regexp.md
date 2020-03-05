# 正则表达式

### 基础

1， `[]`中括号内的对应一个匹配，字符串的任一个可以匹配中括号内的一个字符串。

```javaScript
let reg1 = /[abc]/g
let str1 = 'ab2'
console.log(str1.match(reg1))// [a, b]
```

2，`[^xx]` ^ 表示取反，则表示不匹配中括号内，但是字符串内不在中括号内可以匹配

```javaScript
let reg1 = /[^abc]/g
let str1 = 'ab2'
console.log(str1.match(reg1)) // [2]
```

3, 预定义：

|字符|等于|描述|
|.|[^\n\r]|除了换行与回车之外的任意字符|
|\d|[0-9]|数字|
|\D|[^0-9]|非数字|
|\s|[\t\n\f\r\x0B]|空白|
|\S|[^\t\n\f\r\x0B]|非空白|
|\w|[a-zA-Z0-9]|单词字符|
|\W|[^a-zA-Z0-9]|非单词字符|

4，量词

|代码|描述|
|?|零次或者1次|
|*|零次或者任意次|
|+|1次或者任意次|
|{n}|0次或者n次|
|{n,m}|至少n次，至多m次|
|{n, }|至少n次|

5，分组。小括号`()`表示作为一个单位去进行匹配

6，反向引用。可以通过`Regexp.$1`与`\1`这样的形式去获取。

```javaScript
let reg1 = /(abc)\1/g
let str1 = 'abcabc'
console.log(str1.match(reg1)) // [abcabc]
console.log(Regexp.$1) // abc
```

7, 候选。管道符目标。

8，非捕获性分组. `?:`来表示虽然匹配到分组，但是并没有引用。

```javaScript
let reg1 = /#(?:abc)/g
let str1 = '#abc'
console.log(str1.match(reg1)) // [#abc]
console.log(Regexp.$1) // ''
```

9，前瞻。

|正则|名称|描述|
|(?=exp)|正向前瞻|匹配exp前面的位置|
|(?!exp)|负向前瞻|匹配后面不是exp的位置|
|(?<=exp)|正向后瞻|匹配exp后面的位置|
|(?<!exp)|正向后瞻|匹配exp后面不是exp的位置|

```javaScript
let reg1 = /bed(?=room)/
let str1 = "bedroom"
console.log(str1.match(reg1)) // [bed]

let reg1 = /bed(?!room)/
let str1 = "bedmoor"
console.log(str1.match(reg1)) // [bed]
```

10, 边界

|正则|名称|描述|
|^|开头|不能跟左中括号后面，否则表示不匹配|
|$|结尾||
|\b|单词边界||
|\B|非单词边界||

11, 属性

|属性|描述|
|global|全局|
|ignoreCase|如果创建正则时，设置了i属性，则Regexp.ignoreCase返回true，否则返回false|
|lastIndex|~|
|multiLine|如果设置了m标识，Regexp.multiLine返回true|
|source|返回创建正则的表达式文本|
|sticky|粘性搜索，指定从某个位置开始搜索，设置lastindex|

### 应用

1， 移除所有标签，只剩innerhtml。

> 思路：其实也是移除所有标签，那就只剩内容。

```javaScript
let reg1 = /<(.|\s)*?>/g // 这个地方一定要有问号，表示惰性匹配，否则整个就是贪婪匹配，会匹配到字符串。而？表示先匹配第一个<p>，然后再去匹配
let str1 = "<p><a href='http://www.baidu.com/'>Tesd</a>by <em>黑</em></p>"
console.log(str1.replace(reg1, ''))
```

2，移除所有hr以外的标签，只保留innerhtml

```javaScript
let reg1 = /<(?!hr)(.|\s)*?>/g // （？！）放前面 表示遇到hr就不匹配跳过，如果放后面，则会匹配hr前面的，后面的不会匹配
let str1 = "<p><a href='http://www.cnblogs.com/rubylouvre/'>RRRRR</a></p><hr/><p>by <em>正则</em></p>"
console.log(str1.replace(reg1, ''))
```

3，实现首字符大小

```javaScript
String.prototype.ff = function() {
    let reg = /^\w/
    let self = this
    return self.replace(reg, function(s, index, str) {
        return s.toUpperCase()
    })
}
```

4，匹配表达式中最内侧小括号即小括号内容

```javaScript
let reg1 = /\([^()]+\)/g // 也可以为/\([^\(\)]+\)/g ，这个地方的+,表示必须不匹配
let str1 = '-1-2*((60-30+(-40/5)*(9-2*5/3+7/3*99/4*2998+10*568/14))-(-4*3)/(16-3*2))'
console.log(str1.match(reg1)) // ["(-40/5)", "(9-2*5/3+7/3*99/4*2998+10*568/14)", "(-4*3)", "(16-3*2)"]


```

5，匹配小阔内的内容，不包括括号

```javaScript
let reg3 = /[^()]+(?=\))/g // 也可以为/[^\(\)](?=\))/g， 但是不能/[^()].+(?=\))/g 因为.+ 会匹配到x(1
let str3 = 'x(1)'
console.log(str3.match(reg3)) // ['1']
```

6，匹配第一个乘法或者除法

```javaScript
let reg1 = /\d+[\*\/]\d+/g
let str1 = '9-2*5/3+7/3*99/4*2998+10*568/14'
console.log(str1.match(reg1))
```
7，移除字符串前后空格

```javaScript
let reg1 = /^\s*|\s*$/g
let str1 = ' 撒大声地 收到     '
console.log(str1.replace(reg1, ''))
```

8，匹配日期：

```javaScript
let reg1 = /^(\d{4})[.\-](\d{2})\2(\d{2})$/g // 这个地方匹配第二个-或者。的时候必须用引用，否则发生匹配到第一个-。第二个为。
let str1 = '2020-03-10'
let str1 = '2020.03.10'
console.log(str1.match(reg1))
```

9，括号的界定

> 要求匹配"1231231233" 或者"1451451455"

```javaScript
let reg1 = /^((\d)(\d(\d)))\1\2\3\4$/ // 反向引用时，\1\2\3\4以出现的左括号为准
let str1 = '1231231233'
console.log(str1.match(reg1)) //
```

### 常用：

邮件： ` /^[\w_-]+@[\w_-]+\.[\w_-]+$/`

整数或者小数：`/-?\d+\.?\d+/`
