# Interface

> interface 定义接口时name的第一个字母要大写并且不能为I。

> 4，promise
> 6，类型断言
> 7，接口是一个对象或者数组，每个值又是一个对象指定类型
> 8，接口继承时，若同一值类型不一样
> 9，混合类型
> 12，嵌套
> 13，keyof
> 14，strictNullChecks
> 15，函数带有参数时

### 接口额外属性

正常情况下，参数在受接口约束后，会严格一对一校验，如：

```typescript
interface SquareConfig {
    color : string;
    width: String;
}
```

那么`const a:SquareConfig = { //todo}` 会严格要求a必须严格的有且只有color与width两个属性，如果a有一个height的属性则会报错，或者有拥有额外的属性也会报错。如果要兼容其他属性，接口如下声明：

```typescript
interface SquareConfig {
    color? : string;
    width: String;
    [propName: string]: any;
}
```

还有一种方法是，赋值给中间变量。

```typescript
interface Foo {
  bar?: number;
  bas: string;
}
function mk(params: Foo) {
  console.log(params);
}
const params = { bark: 1, bas: '23' }
mk(params);
```

还可以使用类型断言：

```typescript
mk({ bark: 1, bas: '23' } as Foo);
```

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

### 类的interface

在对类进行检查时，只会对实例部分进行检查，静态部分（constructor）则不会进行检查。

### interface与type区别

type可以理解为别名，在判断时可以使用type判断：

```typescript
type Name = string; // 基本类型
type NameFun = () => string; // 函数
type NameOrRFun = Name | NameFun; // 联合类型
function getName(n: NameOrRFun): Name {
  if (typeof n === 'string') {
    return n;
  }
  return n();
}
```

1，扩展时语法不一样，`interface`是可通过关键字`extend`来实现，`type`是通过`&`。两者可以互相扩展。

2，`type`可以`interface`不可：

```typescript
// 基本类型
type Name = string
// 联合类型
interface Dog {
  wong();
}
interface Cat {
  miao();
}
type Pet = Dog | Cat; // 则至少拥有wong与miao方法一个。
type Pet = Dog & Cat; // 则需要拥有wong与miao两个方法。

type List = [Dog, Cat] // type list的数组第一个 第二个元素分别是对象，每个对象分别拥有wong与miao方法。

```

3，`interface`可`type`不可：

```typescript
// 合并声明
interface User = {
  name: string
}
interface User = {
  sex: number
}
// 等价于
interface User = {
  name: string,
  sex: number
}
```

interface是接口，应该被继承约束，type是变量类型。




```javascript
// interface User {
//   avatar: string;
//   username: string;
//   password: string;
//   gender: number;
//   // 更多的其他属性
// }
// const globalUser: User = {
//   avatar: 'https://www',
//   username: 'foo',
//   password: 'bar',
//   gender: 0
// };
// // 想要更新 globalUser 的部分属性
// function func<K extends keyof User>(incUser: Pick<User, K>) {
//   Object.keys(incUser).forEach((prop: string) => {
//     const key = prop as K;
//     globalUser[key] = incUser[key];
//   });
// }
// func({ username: 'new' });
```
