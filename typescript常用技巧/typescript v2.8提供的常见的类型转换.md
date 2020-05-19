### typescript v2.8提供的常见的类型转换



TS提供了几种内置的预定义的条件类型

- Exclude<T, U> - 用于从类型T中去除不在U类型中的成员
- Extract<T, U> - 用于从类型T中取出可分配给U类型的成员
- NonNullable<T> - 用于从类型T中去除undefined和null类型
- ReturnType<T> - 获取函数类型的返回类型
- InstanceType<T> - 获取构造函数的实例类型
- Required<T> - 用于将类型T的所有属性为required

## Exclude<T, U>

```typescript
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
```

## Extract<T, U> 

```typescript
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void
```

## NonNullable<T> 

```typescript
type T0 = NonNullable<string | number | undefined>;  // string | number
type T1 = NonNullable<string[] | null | undefined>;  // string[]
```

## ReturnType<T>

```typescript
type T0 = ReturnType<() => string>;  // string
type T1 = ReturnType<(s: string) => void>;  // void
type T2 = ReturnType<(<T>() => T)>;  // {}
```

## InstanceType<T> 

```typescript
class C {
    x = 0;
    y = 0;
}

type T0 = InstanceType<typeof C>;  // C
type T1 = InstanceType<any>;  // any
type T2 = InstanceType<never>;  // any
type T3 = InstanceType<string>;  // Error
type T4 = InstanceType<Function>;  // Error
```

## Required<T> 

```typescript
interface Props {
    a?: number;
    b?: string;
};

const obj: Props = { a: 5 }; // OK

const obj2: Required<Props> = { a: 5 }; // Error: property 'b' missing
```

