### compose 函数的实现

目的： d(c(b(a(...args)) -> compose(a, b, c, d)(...args)
实现思路： 遍历所有参数，将其转变为嵌套形式，可以将参数转为数组后使用 reduce 实现。

```javascript
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg;
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return funcs.reduce((a, b) => (...args) => b(a(...args)));
}
```

另外一种写法

```javascript
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg;
  }

  if (funcs.length === 1) {
    return funcs[0];
  }

  return (...args) => funcs.reduce((acc, cur) => cur(acc), ...args);
}
```

两种写法结果都是相同的