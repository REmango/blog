###  keyof（Index types）的使用 

### 1. **最简单的使用，创建一个索引类型**

```typescript
interface Person {
    name: string;
    age: number;
}
let personProps: keyof Person; // 'name' | 'age'
```

`keyof Person`是完全可以与 `'name' | 'age'`互相替换的。但是如果我们在Person上添加address: string时，那么 `keyof Person`会自动变为 `'name' | 'age' | 'address'`。这样相比于我们列举类型，keyof扩展性更强。

### 2.  **索引访问操作符** T[K]

```typescript
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
```

这里我们让函数结果返回为T[K]，帮我们判断key是否取值正确。

```typescript
let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```

### 3. **转换字段创建新的type**

比如我们得到下面两个类型

```typescript
interface PersonPartial {
    name?: string;
    age?: number;
}
interface PersonReadonly {
    readonly name: string;
    readonly age: number;
}
```

使用keyof创建

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

实现Pick类截取

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

使用

```typescript
// 相当于: type PickUser = { age: number; }
type PickPerson = Pick<Person,  "age">
```

上面的Readonly、Partial、Pick在typescript已经内置，可以直接使用



## Record<T, K>

构造一个类型，其属性名的类型为`K`，属性值的类型为`T`。这个工具可用来将某个类型的属性映射到另一个类型上。

```typescript
interface PageInfo {
    title: string;
}

type Page = 'home' | 'about' | 'contact';

const x: Record<Page, PageInfo> = {
    about: { title: 'about' },
    contact: { title: 'contact' },
    home: { title: 'home' },
};
```

