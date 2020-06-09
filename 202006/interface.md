# Interface

> interface 定义接口时name的第一个字母要大写并且不能为I。

> 2，泛型
> 4，promise
> 6，类型断言
> 7，接口是一个对象或者数组，每个值又是一个对象指定类型
> 8，接口继承时，若同一值类型不一样
> 9，混合类型
> 10，与type区别

### 接口校验部分参数

正常情况下，参数在受接口约束后，会严格一对一校验，如：

```typescript
interface SquareConfig {
    color : string;
    width: String;
}
```

那么`const a:SquareConfig = { //todo}` 会严格要求a必须严格的有且只有color与width两个属性，如果a有一个height的属性则会报错，或者有拥有额外的属性也会报错。如果要兼容其他属性，有两种办法，第一种是接口如下声明：

```typescript
interface SquareConfig {
    color? : string;
    width: String;
    [propName: string]: any;
}
```

第二种是借助类型断言。

### 类型断言

类型断言是编译时的语法，会覆盖默认的类型判断。

```typescript
interface Foo {
  bar: number;
  bas: string;
}

const foo = {} as Foo; // 若不通过as Foo做类型断言，则认为{}是一个没有属性的空对象，那么后续添加k-v就会报错
foo.bar = 123;
foo.bas = 'hello';
```

除了通过`as Foo`进行类型断言，还可以通过`<string>foo`的形式：

```typescript
interface Foo {
  bar: number;
  bas: string;
}

const foo = <Foo>{};
foo.bar = 123;
foo.bas = 'hello';
```

不过这样的形式会与jsx组件语法一样，所以不建议此种写法。

如果断言内容与接口一方为另外一方的子集时，可以断言成功。但是如果类型不同时，可以通过any来实现双重断言。

```typescript
(event as any) as Element
```

### 类的interface？？

在对类进行检查时，只会对实例部分进行检查，静态部分（constructor）则不会进行检查。
