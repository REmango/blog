### type与interface的区别

此文是主要是对官网[Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases)的总结

## 先看看type的定义

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。

```typescript
type Name = string; // 相当于string
type NameResolver = () => string; // 数组
type NameOrResolver = Name | NameResolver; // 作用于联合类型
```

起别名不会新建一个类型 - 它创建了一个新 *名字*来引用那个类型。别名可以让我们更好的去使用或和复用

## 区别

###  1. 定义上

Interface：创建了一个新的名字，可以在其它任何地方使用

Type: 并不创建新名字, 别名的含义是指它不是真正的类型。同时type在写法上是使用赋值符号“=”，而interface是直接在后面加{}。

在下面的示例代码里，在编译器中将鼠标悬停在 `interfaced`上，显示它返回的是 `Interface`，但悬停在 `aliased`上时，显示的却是对象字面量类型。

```typescript
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```

### 2. typeof创建type

type 语句中还可以使用 typeof 获取实例的 类型进行赋值

```typescript
// 当你想获取一个变量的类型时，使用 typeof
let div = document.createElement('div');
type B = typeof div
```

### 3. extends和implements

interface可以使用extends和implements，而type没有。

那么type想达到类型合并，怎么办呢？如下

```typescript
type Name = { 
  name: string; 
}
type User = Name & { age: number  };
```

type也可以合并interface

```typescript
interface Name { 
  name: string; 
}
type User = Name & { 
  age: number; 
}
```

总结：

 因为 [软件中的对象应该对于扩展是开放的，但是对于修改是封闭的](https://en.wikipedia.org/wiki/Open/closed_principle)，你应该尽量去使用接口代替类型别名