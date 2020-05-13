## 为什么要使用 react-hook

react-hook 是在 v16.8 版本引入的。它是一个增强版的函数式组件，可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

### 之前版本的函数式组件

函数式组件

```javascript
const App = props => <div>Hello, {props.name}</div>;
```

类组件

```javascript
class App extends React.Component {
  render() {
    return <div>Hello, {this.props.name}</div>;
  }
}
```

我们可以看出函数式组件代码更为简洁，当业务复杂，组件庞大的时候，函数式组件更加方便维护。

此外，函数式组件会表现更好的性能。因为类组件至少会有一个继承和调用 constructor 的行为。所以能用函数式组件就用函数式组件。

缺点：此时的函数式必须是纯函数，不能包含状态，也不支持生命周期方法，因此无法取代类。只适用于少部分的情况。

### 类组件缺陷

#### 1. this 的绑定

这个不是 react 本身的问题，而是由于 class 中 this 的指向导致。

```javascript
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
    	name: 'zhangsan',
    },
    this.handler = this.handler.bind(this);
  }
  handler () {
    this.setState({
      name: 'lisi',
    })
  }
  render() {
    <div >
      <button onclick= {this.handler}><button/>
    </div>
  }
}
```

### 2. 复杂的生命周期

componentWillMount、componentWillReceiveProps、componentWillUpdate 这 3 个生命周期将在 v17 版本删除，原因是它们是不安全的，使用者滥用很容易造成 bug。

v16.4 新增了 getDerivedStateFromProps、getSnapshotBeforeUpdate 以替代上述 3 个生命周期。

react 但是为了版本兼容，在 v16.3 以后的 v16.x 版本中，新旧的方法依旧都可以使用。

具体原因见 [Update on Async Rendering](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)

随着业务逻辑复杂度的增加，我们的组件经常会在一个或多个生命周期中做很多时，造成后期维护困难。

### 3.性能

class 组件在使用 babel 编译是相比于函数式体积大很多。

### 结论

> 所以，如果函数式组件如果能够具有 state 和生命周期（钩子函数），那该是多么好的事情啊。react-hook 由此诞生

### react-hook 的简单使用

#### 1.useState()

```javascript
import React, { useState } from 'react';

function Example() {
  // 声明一个新的叫做 “count” 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

用户点击按钮，会导致按钮的文字改变，文字取决于用户是否点击，这就是状态。

#### 2.useEffect()

useEffect 具有两个参数，可以实现 componentDidMount，componentDidUpdate， componentWillUnmount 三个生命周期。

```javascript
useEffect(() => {
  // Async Action
}, [dependencies]);
```

第一个参数是一个函数，异步操作的代码放在里面。第二个参数是一个数组，用于给出 Effect 的依赖项，只要这个数组发生变化，`useEffect()`就会执行。第二个参数可以省略，这时每次组件渲染时，就会执行`useEffect()`。

```javascript
const Person = ({ personId }) => {
  const [loading, setLoading] = useState(true);
  const [person, setPerson] = useState({});

  useEffect(() => {
    setLoading(true);
    fetch(`https://swapi.co/api/people/${personId}/`)
      .then(response => response.json())
      .then(data => {
        setPerson(data);
        setLoading(false);
      });
  }, [personId]);

  if (loading === true) {
    return <p>Loading ...</p>;
  }

  return (
    <div>
      <p>You're viewing: {person.name}</p>
      <p>Height: {person.height}</p>
      <p>Mass: {person.mass}</p>
    </div>
  );
};
```

每当组件参数`personId`发生变化，`useEffect()`就会执行。组件第一次渲染时，`useEffect()`也会执行。

问：如果只想执行一次怎么办？

答：将第二个参数设置为[]即可。

### 其他 hook

详情请看官网示例 [Hook API 索引](https://zh-hans.reactjs.org/docs/hooks-reference.html)

### 参考链接

[React Hooks 入门教程](https://www.ruanyifeng.com/blog/2019/09/react-hooks.html)

[Why React Hooks](https://juejin.im/post/5c6971f96fb9a049fb443459)
