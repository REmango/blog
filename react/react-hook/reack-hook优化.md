# react-hook优化

## state相同时的重复渲染优化

这里，我们使用react-hook编写一个简单的计数组件

```jsx
import React, { useState, useRef } from "react"

export default function Counter() {
  const [counter, setCounter] = useState(0)
  const renders = useRef(0)

  return (
    <div>
      <div>Counter: {counter}</div>
      <div>Renders: {renders.current++}</div>
      <button onClick={() => setCounter(counter + 1)}>Increase Counter</button>
    </div>
  )
}
```

多次按“增加计数器”按钮时，渲染计数器将显示与计数器状态完全相同的数字。这意味着`Counter`只要我们的计数器状态更改，组件就会更新。

但是，当您输入`App`组件文本输入时，您会看到渲染计数器也增加了。这意味着`Counter`只要文本输入状态发生更改，我们的组件就会重新呈现。

为了避免setState值相同引起的重复渲染，我们可以将state缓存起来，当相同时则不渲染。`React.memo`为此而诞生。

```jsx
import React, { useState, useRef } from "react"

export default React.memo(() => {
  const [counter, setCounter] = useState(0)
  const renders = useRef(0)

  return (
    <div>
      <div>Counter: {counter}</div>
      <div>Renders: {renders.current++}</div>
      <button onClick={() => setCounter(counter + 1)}>Increase Counter</button>
    </div>
  )
})
```

## 父组件传入的props相同时的重复渲染优化

在下面的组件中，只要改组件渲染，Couter组件一定渲染。但是，只要couter的props没有变化，Couter的渲染是没有必要的。

```javascript
import React, { useState } from "react"
import Counter from "./Counter"

export default function App() {
  const [value, setValue] = useState("")

  return (
    <div>
      <input
        type="text"
        onChange={e => setValue(e.target.value)}
        value={value}
      />
      <Counter
        addHello={() => setValue(value + "Hello!")}
        myObject={{ key: "value" }}
      />
    </div>
  )
}
```

我们可以使用useCallback或useMemo将传入的props或值缓存起来，避免重复渲染。

```jsx
import React, { useState, useCallback } from "react"
import Counter from "./Counter"

export default function App() {
  const [value, setValue] = useState("")
  const [newValue, setNewValue] = useState("")

  const addHello = useCallback(() => setValue(value + "Hello!"), [value])
  const myObject = useMemo(() => ({ key: "value" }), [])

  return (
    <div>
      <input
        type="text"
        onChange={e => setValue(e.target.value)}
        value={value}
      />
      <input
        type="text"
        onChange={e => setNewValue(e.target.value)}
        value={newValue}
      />
      <Counter addHello={addHello} myObject={myObject} />
    </div>
  )
}
```

## hook中的批量状态更新

在class组件中，this.setState支持状态批量更新

```javascript
this.setState(() => ({
  setData: [],
  loading: false
}))
```

但是在hook中略有不同，你可以这样这样做。

```javascript
function ClickButton(props) {
  const [loading, setLoading] = useState(false);
  const [data, setData] = useState([]);


  const onClickCount = () => {
    setLoading(true)
    Promise.resolve(A).then((data) => {
      setLoading(false)
      setData([])
    })
  };

  return (
    <div>
      <button onClick={onClickCount}>Counter</button>
    </div>
  );
}
```

问题: 我们知道在setTimeout，或是promise这样的异步函数中setState是无法设置批量更新的，所以上述代码异步promise的返回结果会造成两次组件的更新。

解决方案：

1. unstable_batchedUpdates

```javascript
ReactDOM.unstable_batchedUpdates(() => {
  setLoading(false)
  setData([])
})
```

2. useReducer

useReducer是useState的替代方案，使我们可以使用一个全局的state对象保存所以的状态，而不是一个值，类似于redux。

```javascript
const initialState = {loading: false, data: []};

function reducer(state, action) {
  switch (action.type) {
    case 'loadingRun':
      return {...state, loading: ture};
    case 'request':
      return {loading: false, data: action.data};
    default:
      throw new Error();
  }
}

  const onClickCount = () => {
    dispatch({type: 'loadingRun' })
    Promise.resolve(A).then((data) => {
      dispatch({type: 'request', data })
    })
  };

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={onClickCount}>-</button>
    </>
  );
}
```

