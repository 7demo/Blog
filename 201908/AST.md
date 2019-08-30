# AST，与JavaScript编译

## AST概念

> AST, Abstract Syntax Tree, 是抽象语法树。将源程序代码进行词法分析，转成程序语法树。

一般情况下，有以下三阶段：

+ 词法分析，又称线性扫描、线性分析。

 > 它逐个读取源程序的字符，把他们组成词法记号(token)流。（可以在https://esprima.org/demo/parse.html#查看。）

 比如输入为：

 ```javascript
 var answer = 6 * 7;
 ```

 解析成`token`即为：

 ```
 [
    {
        "type": "Keyword",
        "value": "var"
    },
    {
        "type": "Identifier",
        "value": "answer"
    },
    {
        "type": "Punctuator",
        "value": "="
    },
    {
        "type": "Numeric",
        "value": "6"
    },
    {
        "type": "Punctuator",
        "value": "*"
    },
    {
        "type": "Numeric",
        "value": "7"
    },
    {
        "type": "Punctuator",
        "value": ";"
    }
]
 ```

+ 语法分析，又叫层次分析。是token记号流根据语法结构进行层次分组，形成语法短语，语法短语常用分析树表示。

+ 语义分析，生成语法树。之后还可以有代码生成、代码优化阶段。

## javascript编译过程

+ 词法分析。

+ 语法分析，生成语法树。比如：

```javascript
const a = 1
console.log(a)
```

生成的语法树为：

```
{
  "type": "Program",
  "start": 0,
  "end": 26,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 11,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 6,
          "end": 11,
          "id": {
            "type": "Identifier",
            "start": 6,
            "end": 7,
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "start": 10,
            "end": 11,
            "value": 1,
            "raw": "1"
          }
        }
      ],
      "kind": "const"
    },
    {
      "type": "ExpressionStatement",
      "start": 12,
      "end": 26,
      "expression": {
        "type": "CallExpression",
        "start": 12,
        "end": 26,
        "callee": {
          "type": "MemberExpression",
          "start": 12,
          "end": 23,
          "object": {
            "type": "Identifier",
            "start": 12,
            "end": 19,
            "name": "console"
          },
          "property": {
            "type": "Identifier",
            "start": 20,
            "end": 23,
            "name": "log"
          },
          "computed": false
        },
        "arguments": [
          {
            "type": "Identifier",
            "start": 24,
            "end": 25,
            "name": "a"
          }
        ]
      }
    }
  ],
  "sourceType": "module"
}
```

+ 预编译。js相比其他静态语言，多了一个预编译步骤。这个是发生在执行之前。在预编译中主要是进行代码检查与优化。

# 并且这个步骤中，编译器会找到所有变量并进行提升（赋值发生在执行阶段），提升规则是：变量与函数声明进行提升，函数声明优先级大于变量提升，后面的函数声明会覆盖前面的变量声明；函数表达式不进行提升。#

+ 解释执行。

## ASI

> ASI, automatic semicolon insertion， 指分号自动插入，在程序解析阶段，在某些语句没有分号情况下，判断语句结束，来自动插入分号，认为语句结束，进行解析。

某些语句是不需要分号的，如果写上分号，会被认为空语句。

```javascript
function a(){

}; // 多余分号
```

需要明确的是，ASI是第二选择。默认情况下，在遇到行结束符时，会忽略它，如果解析不正确，才会把行结束符认为是语句结束符。比如：

```javascript
var a = 1
var b = 2
```

会把`var a = 1 \n var b = 2`先解析`var a = 1 var b = 2`，遇到解析错误`Uncaught SyntaxError: Unexpected token var`, 才会解析成两行。所以从ASI的角度，分号是不能省略的。

不过，在ECMASCRIPT中，有个”限制生产“，是指如果语句有结束符，parser会优先认为结束符为语句的结束。比如：`return`。

```javascript
function a() {
    return
    {};
}
a()   //
```

其他还有：`continue`、`break`、`throw`、箭头函数、`yield`与后自增自减。

当然，`for`循环总不是用`ASL`。

## LHS/RHS

在js中，`LHS`表示赋值操作，`RHS`表示读取值操作。

```javascript
console.log(a) // RHS
var a = 2 // LHS
```

> 都说“学会AST, 可以为所欲为”，其实babel、eslint、webpack、rollup等都使用到了`AST`。