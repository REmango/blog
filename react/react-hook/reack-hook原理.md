

#  react-hook原理

>本文并不是原创，主要是对[React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e), [Under the hood of React’s hooks system](https://medium.com/the-guild/under-the-hood-of-reacts-hooks-system-eb59638c9dba)两文的归纳总结。

hook让我们可以在函数组件中实现类组件生命周期和状态，同时在不暴露实例的情况下（不需要使用 `this` 关键字）管理组件。虽然无法做到完全替代，但是能让我们解决大部分的需求，给我们提供了便捷的开发。

但是react-hook使用不当很容易出现bug并且调试非常困难。所以，明白其原理能够帮助我们更好的使用它，帮助我们避免错误的发生。

## hook使用事项

> - **Don’t call Hooks inside loops, conditions, or nested functions**
> - **Only Call Hooks from React Functions**

翻译过来就是不能在循环、判断内部使用 Hook，只能在React函数组件内部使用它。

### 原理(主要以useState为例)

> not magic, just arrays

先让我们回顾下使用useState的代码

```javascript
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState("Rudi");
  const [lastName, setLastName] = useState("Yardley");

  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```

当调用 useState 的时候，会返回形如 `(变量, 函数)` 的一个元祖。我们通过解构获取相应的状态以及setState，useState的参数即为state的初始值。

简单实现

```javascript
function render() {
  ReactDOM.render(<App />, document.getElementById("root"));
}

let state: any;

function useState<T>(initialState: T): [T, (newState: T) => void] {
  state = state || initialState;

  function setState(newState: T) {
    state = newState;
    render();
  }

  return [state, setState];
}

render(); // 首次渲染


```

下面我们根据图详细看看具体的实现

###  1.初始化

![](https://miro.medium.com/max/1280/1*LAZDuAEm7nbcx0vWVKJJ2w.png)

首先创建两个空的数组，'state'和 'setters'

### 2. 首次渲染

第一次渲染时候，根据 useState 顺序，逐个声明 state 并且将其放入全局 Array 中。每次声明 state，都要将 cursor 增加 1。

如下图所示

![](https://miro.medium.com/max/1260/1*8TpWnrL-Jqh7PymLWKXbWg.png)



### 3. 更新



更新 state，触发再次渲染的时候。**cursor 被重置为 0**。按照 useState 的声明顺序，依次拿出最新的 state 的值，视图更新。

```javascript
states[currenCursor] = states[currenCursor] || initialState; // 检查是否渲染过
```

![img](https://miro.medium.com/max/627/1*qtwvPWj-K3PkLQ6SzE2u8w.png)

### 4. 代码简单实现



```javascript
let state = [];
let setters = [];
let firstRun = true;
let cursor = 0;

function createSetter(cursor) {
  return function setterWithCursor(newVal) {
    state[cursor] = newVal;
  };
}

// This is the pseudocode for the useState helper
export function useState(initVal) {
  if (firstRun) {
    state.push(initVal);
    setters.push(createSetter(cursor));
    firstRun = false;
  }

  const setter = setters[cursor];
  const value = state[cursor];

  cursor++;
  return [value, setter];
}

// Our component code that uses hooks
function RenderFunctionComponent() {
  const [firstName, setFirstName] = useState('Rudi'); // cursor: 0
  const [lastName, setLastName] = useState('Yardley'); // cursor: 1

  return (
    <div>
      <Button onClick={() => setFirstName('Richard')}>Richard</Button>
      <Button onClick={() => setFirstName('Fred')}>Fred</Button>
    </div>
  );
}

// This is sort of simulating Reacts rendering cycle
function MyComponent() {
  cursor = 0; // resetting the cursor
  return <RenderFunctionComponent />; // render
}

console.log(state); // Pre-render: []
MyComponent();
console.log(state); // First-render: ['Rudi', 'Yardley']
MyComponent();
console.log(state); // Subsequent-render: ['Rudi', 'Yardley']

// click the 'Fred' button

console.log(state); // After-click: ['Fred', 'Yardley']

```

### 不能在循环、判断内部使用 Hook

如果使用了会怎么样呢？

让我们看看下面的例子：

```javascript
let firstRender = true;

function RenderFunctionComponent() {
  let initName;
  
  if(firstRender){
    [initName] = useState("Rudi");
    firstRender = false;
  }
  const [firstName, setFirstName] = useState(initName);
  const [lastName, setLastName] = useState("Yardley");

  return (
    <Button onClick={() => setFirstName("Fred")}>Fred</Button>
  );
}
```







### 参考链接

[React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)

[Under the hood of React’s hooks system](https://medium.com/the-guild/under-the-hood-of-reacts-hooks-system-eb59638c9dba)

[一文彻底搞懂react hooks的原理和实现](https://juejin.im/post/5daee8b7e51d4524ce222825)