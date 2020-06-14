# react-hook注意事项

## 1.  异步中获取state

有这样一种情形，见下面代码。当我们不断使用hook中setState触发状态的更新，在我们异步获取状态时，getCount获取的状态时1s前的状态，而不是现在的状态。这是为什么呢？

```javascript
function ClickButton(props) {
  const [count, setCount] = useState(0);
  const onClickCount = () => {
    setCount((c) => c + 1);
  };
  const getCount = () => {
    setTimeout(() => {
      console.log(count)
    }, 1000)
  };
  return (
    <div>
      <button onClick={onClickCount}>Counter</button>
      <button onClick={getCount}>Submit</button>
    </div>
  );
}
```

导致异步获取不到count的原因是闭包导致的，当异步函数创建时，即对count的值进行了保存，是函数的正常行为。解决方案是，将变量从 `值`变为`引用`，这样我们就可以获取通过对象获取及时状态。

如下

```javascript
function ClickButton(props) {
  const count = useRef(0);
  const onClickCount = () => {
    count.current++;
  };
  const getCount = () => {
    setTimeout(() => {
      console.log(count.current)
    }, 1000)
  };
  return (
    <div>
      <button onClick={onClickCount}>Counter</button>
      <button onClick={getCount}>Submit</button>
    </div>
  );
}
```

上面的setTimeout保存的count是一个对象。

## 2. hook中的批量状态更新

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



## 3.避免useEffect的滥用

React 引入的最好用，最贴心的一个 hook 是 useEffect。它可以处理与 prop 或 state 更改相关的动作。可就算它很好用，人们也不该到处滥用它。

想象一下有一个组件，其获取一个项目列表并将其渲染给 dom。另外，如果请求成功，我们将调用“onSuccess”函数，该函数作为一个 prop 传递给这个组件。

<center>这很危险</center>

```javascript
function DataList({ onSuccess }) {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);
  const fetchData = useCallback(() => {
    setLoading(true);
    callApi()
      .then((res) => setData(res))
      .catch((err) => setError(err))
      .finally(() => setLoading(false));
  }, []);
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  useEffect(() => {
    if (!loading && !error && data) {
      onSuccess();
    }
  }, [loading, error, data, onSuccess]);
  return <div>Data: {data}</div>;
}
```

一共有两个 useEffect hooks，第一个在初始渲染时处理 api 调用，第二个会调用 onSuccess 函数，假设当状态没有加载、没有错误但有数据时调用肯定成功。这很有道理是吧？

对第一个调用来说这肯定是正确的，并且可能永远不会失败。但你也失去了动作和需要调用的函数之间的直接联系。同样也没有 100％的保证可以说这种情况仅在 fetch 动作成功后才会发生，而这正是我们开发人员不想看到的。

<center>解决方案</center>

```javascript
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);
  const fetchData = useCallback(() => {
    setLoading(true);
    callApi()
      .then((fetchedData) => {
        setData(fetchedData);
        onSuccess();
      })
      .catch((err) => setError(err))
      .finally(() => setLoading(false));
  }, [onSuccess]);
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  return <div>{data}</div>;
}
```

现在一目了然了，在 api 调用成功的情况下才调用 onSuccess。

**单一责任的useEffect**

还记得以前，我们只能用 componentWillReceiveProps 或 componentDidUpdate 方法挂接到 React 组件的渲染过程吗？那是一段黑暗的回忆，也让我们意识到了 useEffect hook 的美妙之处，尤其是你可以随意使用这些 hooks。

但是有时因为粗心而让“useEffect”身兼数职，就会带回那些黑暗的回忆。例如，假设你有一个组件以某种方式从后端获取一些数据，并且还会根据当前位置显示面包屑。（再次使用 react-router 获取当前位置。

```javascript
function Example(props) {
  const location = useLocation();
  const fetchData = useCallback(() => {
    /* Calling the api */
  }, []);
  const updateBreadcrumbs = useCallback(() => {
    /* Updating the breadcrumbs*/
  }, []);
  useEffect(() => {
    fetchData();
    updateBreadcrumbs();
  }, [location.pathname, fetchData, updateBreadcrumbs]);
  return (
    <div>
      <BreadCrumbs />
    </div>
  );
}
```

这里有两个用例，即“数据获取”和“显示面包屑”。两者都通过 useEffect hook 更新。当 fetchData 和 updateBreadcrumbs 函数或 location 更改时，都会运行这个 useEffect hook。现在的主要问题是，当位置更改时，我们还调用了 fetchData 函数。这可能是我们没有想到的副作用。

```javascript
function Example(props) {
  const location = useLocation();
  const updateBreadcrumbs = useCallback(() => {
    /* Updating the breadcrumbs*/
  }, []);
  useEffect(() => {
    updateBreadcrumbs();
  }, [location.pathname, updateBreadcrumbs]);
  const fetchData = useCallback(() => {
    /* Calling the api */
  }, []);
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  return (
    <div>
      <BreadCrumbs />
    </div>
  );
}
```

