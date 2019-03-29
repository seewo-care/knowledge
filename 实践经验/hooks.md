# React Hooks 

## hooks诞生的原因

在[react的今天和明天](https://juejin.im/post/5be90d825188254b0917f180)系列文章中，react开发人员介绍了class组件存在的三个问题。

- 逻辑复用
- 庞大的组件
- 难以理解的class

在class组件中通过HOC和render props中来实现组件的逻辑复用。这会带来一个问题，当我们拆分出很多细小的组件再将它们组合到一起，如果在chrome打开react的扩展，会发现组件层级非常深，增加了react的计算负担。这个问题称为“包裹地狱（wrapper hell）”。

当我们试图解决第一个问题的时候，我们会将更多的逻辑放在单个的组件中，导致组件日益庞大。这就是第二个问题。

js中的class实际上是function的语法糖。在使用过程中，它隐藏了function的实现细节（static， prototype等）。并且当我们写一个函数组件的时候，如果要添加状态，那么就必须将它转换为class组件，这会写很多样板代码。不仅如此，对于机器来说，压缩后的class的组件中的所有的方法名都是没有经过压缩的。并且也无法tree shaking。这是class组件的第三个问题。

`Dan Abramov`认为这不是三个独立的问题，而是一个问题的三个部分。为了解决这个问题，于是出现了hooks。

## hooks简介

hooks是另一种书写组件的方式，在hooks中只有函数而没有class。通过hooks提供的一系列API几乎可以完全覆盖class组件中的情况（为什么说是几乎？在下面差异部分会提到）。hooks是更加简洁优雅的，逻辑分离的。

- v16.8。hooks是react16.8版本新添加的功能。如果要使用hooks，需要确保react版本升级到16.8.0及以上，同时保证react-dom和react的版本保持一致。

- 百分之百向后兼容

- react没有计划移除class组件。hooks是一种可选的写法，你依然可以选择不用而使用class（用过了你会喜欢上它）。

- hooks中使用的依旧是class中的概念。

## hooks API

hooks有下列API。所有的示例demo都在这里。https://codesandbox.io/s/o968n1q625 ，可以打开并直接看到效果。

### Basic Hooks

- useState
- useEffect
- useContext

### Additional Hooks

- useReducer
- useCallback
- useMemo
- useRef
- useImperativeHandle
- useLayoutEffect
- useDebugValue

接下来会解释这些API并附有详细的demo。

#### 1.useState

useState是hooks中最基础的一个API。其使用方式类似于class中的this.setState，不过又有所不同。下面是demo中的示例。

```
function Counter() {
  const [count, setCount] = useState(4);
  return (
    <>
      <p>
        count: {count}, random: {Math.random()}
      </p>
      <button onClick={() => setCount(count < 5 ? count + 1 : count)}>
        点击这里加1
      </button>
      <button onClick={() => setCount(count - 1)}>点击这里减1</button>
    </>
  );
}
```

`useState(4)`用于创建一个state变量，并传入初始值。返回值是一个数组，通过解构写法拿到返回的值。count是一个可变的值（只能通过setCount改变），它的初始值是useState传入的参数4。单击按钮的时候，通过setCount去改变它的值。从而重新渲染该组件。

它与class组件中的setState的差异有以下几点：

- 不会进行状态的合并，而是进行状态的替换。这在简单类型没有什么问题，对于复杂类型如对象使用的时候需要注意（可以在demo中渲染Count2组件）。
- 当使用setCount改变状态时，使用Object.is算法比较状态的前后值。它和 === 的区别在于对于0和-0的判断，以及NaN的判断。
- 如果前后状态相同，那么组件不会进行渲染。在class组件中使用setState，即使某个状态前后是相同的值，仍会进行渲染（demo中点击按钮让count的值改变到5可以看到）。

另外，如果useState的参数值需要复杂的计算才能得到，那么可以传入给useState一个函数，它仅在首次渲染时执行该复杂的计算函数。`useState(() => expensiveCalc())`。

#### 2.useEffect/useLayoutEffect

由于useEffect和useLayoutEffect具有相同的使用方式，就放到一起来介绍。
正如其名，useEffect用来处理副作用。副作用包括DOM的改变、订阅、定时器、日志等。副作用不允许放到函数体中。

`useEffect/useLayoutEffect`可以取代componentDidMount、componentDidUpdate、componentWillUnmount三个声明周期。所以它是非常强大一个hook，在使用方面很灵活，也不是那么容易理解。

`useEffect`接受两个参数，其中第二个参数是可选的，不过一般情况下都需要传入第二个参数。

```
useEffect(() => {
  // some side effect
  // ...

  // return a clean up function
  return () => {

  }
}, [])
```

第一个参数是一个函数，当组件首次渲染或者其依赖的状态改变时它会执行。该函数的返回值是可选的，可以不写，如果要写的话，必须是一个函数，用于清除上一个状态。

第二个参数是可选的，它是一个数组。数组中可以传入状态值（通过useState产生的值），当状态值改变的时候首先会执行return函数，用于清理上一个状态，然后useEffect中的函数就会再次执行。

- 如果不传入第二个参数，代表组件中任何状态的改变该effect都会执行一次，这通常不是我们想要的行为。
- 如果第二个参数传递一个空数组，代表该effect仅会执行一次，相当于componentDidMount。return函数也只会在组件卸载的时候执行一次，相当于componentWillUnmount。
- 如果第二个参数数组中有一个或多个状态（demo中的useLayEffect），那么只要有任意一个状态值发生变化，该effect都会再次执行。相当于componentDidUpdate。

```
// demo--useLayoutEffect
useEffect(() => {
  if (value.length > 10) {
    setValue(value.substring(0, 10));
  }
  setLengths(value.length);
},[value]);
```

当value发生变化的时候，effect再次执行。改变length状态。

useLayoutEffect的语法和useEffect一样。不同点在于：

- useEffect是在组件状态改变后，并且在组件layout和paint之后，也就是说组件出现在页面后再进行调用。useLayoutEffect是在组件状态改变后，但是在组件layout和paint之前，也就是在组件出现在页面之前进行调用。
- useEffect是异步的，useLayoutEffect是同步的。可以看这篇文章：https://juejin.im/post/5c8f436ff265da610e5ec22e 。

在useLayoutEffect的demo中，尝试将useEffect改变成useLayoutEffect，然后在输入框输入第十一个字符，可以看到明显差别。在大多数情况下你应该使用useEffect。因为useEffect是异步的，不会堵塞主线程渲染。

#### 3.useRef

在16.3版本中，引入React.createRef，来代替字符串ref。同样，在hooks中，useRef也可以取代createRef。可以查看demo中的useRef。

```
function xxx() {
  const inputRef = useRef(null);
  useEffect(() => {
    inputRef.current.value = 'hello';
  }, [])
  return (
    <input type="text" ref={inputRef} />
  )
}
```

useRef接受一个初始值，返回一个可变的ref对象，ref.current指向初始化的值。它可以指向别的值。

另外，由于是函数组件，this不再指向这个组件，所以如果要达到class组件中实例变量的效果，也可以通过useRef来实现。

```
const timerRef = useRef(null);

useEffect(() => {
  timerRef.current = setInterval(() => {
    inputRef.current.value = 'hello';
  }, 1000);
  return () => {
    clearInterval(timerRef.current);
  };
}, []);
```

这里的timerRef.current相当于class中的实例变量。

#### 4.useContext

在React中，如果要将上层的属性传递到下层，一般来说需要一层一层的传递。比如A->B-C->D。Context是一种数据传递机制，用于跨层级传递数据。比如在D组件可以直接使用A组件的数据。可以查看demo中的useContext。

useContext是Context.Consumer(16.3)以及static contextType(16.6)的一种简写。

上层组件定义Context.Provider，并传入value属性。在子组件中通过useContext(Context)可以获取value属性。

```
// parent.js
const parentContext = createContext();
function Parent(props) {
  const countArr = useState({
    count1: 0,
    count2: 1
  });
  return (
    <parentContext.Provider value={countArr}>
      {props.children}
    </parentContext.Provider>
  );
}

function Child() {
  const countArr = useContext(parentContext);
  const [countObj, setCountObj] = countArr;

  return (
    <>
      <div>
        count1: {countObj.count1} count2: {countObj.count2}
      </div>
    </>
  );
}

<Parent><Child /></Parent>
```

可以看到，这里传递属性不是通过props的，而是通过Context的。需要注意的是，在父组件定义了parentContext，需要将其导出，因为子组件使用useContext的参数就是parentContext。

另外，因为没有static的限制，同一个组件中可以使用多个useContext。也就是说，可以使用多个上层组件传递来的数据。

#### 5.useReducer

提到reducer，首先想到的应该是redux中的reducer。useReducer这个hook与redux中的reducer有所相似又有所不同。可以查看demo中的useReducer。

定义一个reducer的方式和redux中是一样的：

```
const initialState = {
  count: 0
};
const reducer = (state = initialState, action) => {
  switch (action.type) {
    case 'ADD': {
      return {
        count: state.count + 1
      };
    }
    case 'MINUS': {
      return {
        count: state.count - 1
      };
    }
  }
};


function Counter1() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      <div>count: {state.count}</div>
      <button onClick={() => { dispatch({ type: 'ADD' }); }}>+</button>
      <button onClick={() => { dispatch({ type: 'MINUS' }); }}>-</button>
    </>
  );
}
```

useReducer接受连个参数，一个是reducer，一个是initialState。返回一组值，分别是state和dispatch。通过dispatch触发一个动作，进而去更新状态。如果你使用过redux，那么理解起来没有任何困难。

另外，与redux中的reducer有所不同的是，useReducer中的reducer是独立的。如果有多个组件使用到了同一个reducer，那么它们之间的状态是独立的。相较于redux的全局共享状态，它还依赖于react-redux提供的Provider组件。

所以是不是突然想到了第四点中的Context，它提供了Provider。如果能配合useReducer，就可以实现全局状态共享了？确实如此！具体的代码可以查看demo中的context-reducer。

还有一点必须需要提到的是，由于性能原因，react-redux没有推出官方的useRedux。 具体原因可以看这篇文章：https://juejin.im/post/5c7c8dece51d45607d5324cd 。在最近的[7.0的beta版本的发布说明](https://github.com/reduxjs/react-redux/releases)中，react-redux团队宣布将在7.x版本中推出useRedux API。

#### 6.useImperativeHandle

在class组件中，如果父组件需要改变子组件的状态，有两种方式。一种是就是通过改变父组件state，该state作为props传给子组件，从而改变子组件的状态。另一种就是通过操作子组件的ref了。传递ref的方式主要有两种，createRef和forwardRef，具体的就不再细说。useImperativeHandle这个hook就是ref的另一种写法。以前是在父组件中拿到子组件元素的ref，直接操作ref代表的元素节点，相当于是直接操作子元素的dom元素。现在通过这个hook可以在子组件中暴露一些API供父组件调用，而父组件是不能直接操作子组件的dom元素的。迪米特法则就是这样描述的：一个类对它所调用的类的细节知道的越少越好。具体代码可以查看demo中的useImperativeHandle。

```
function Child(props) {
  const inputRef = useRef(null);
  useImperativeHandle(props.myref, () => ({
    focus() {
      inputRef.current.focus();
    },
    setValue(value) {
      inputRef.current.value = value;
    }
  }));
  return (
    <>
      <input type="text" ref={inputRef} />
    </>
  );
}
```

不过在实际开发中，你应该尽可能通过传递props来改变子组件，通过ref来改变子组件是一种不推荐的方案。

#### 7.useCallback

useCallback用来缓存一个函数。在函数式组件中可能有这样一种情况，父组件调用子组件，并将一个函数传递给子组件，假设子组件是使用了memo的（如果属性值没有变化，那么将不会重新渲染）。具体代码可以查看demo中的useCallback。

```
function Parent() {
  //...
  const handleChange = () => { // ... }
  return (
    <>
      <p>count: {count}</p>
      <Child onChange={handleChange}/>
    </>
  )
}
```

当父组件中的count状态改变时，这时候父组件重新渲染，子组件尽管没有任何改变，但由于onChange这个属性是一个新的函数，它还是重新渲染了。这是我们不期望的行为。在class组件中我们往往通过`onChange={this.handleChange}`将函数传递给子组件。那么下次父组件渲染时，this.handleChange是没有变化的。如果子组件是memo的，那么子组件将不会重新渲染。

所以uesCallback就是为了解决这样一个问题，它缓存一个函数，并接受一系列依赖项，返回一个函数。如果依赖项没有变化，那么返回的函数不会变化。

```
function Parent() {
  const [count, setCount] = useState(0);
  const [count2, setCount2] = useState(0);
  const [result, setResult] = useState(count);
  useEffect(() => {
    setInterval(() => {
      // setCount(prevCount => prevCount + 1);
      setCount2(prevCount => prevCount + 1);
    }, 1000);
  }, []);

  const handleChange = useCallback(() => {
    setResult(count + 1);
  }, [count]);
  // const handleChange = () => { setResult(count + 1) };

  return (
    <>
      <Counter count={count} />
      <Counter count={count2} />
      <Child onChange={handleChange} />
      <p>result: {result}</p>
    </>
  );
}
```

这里的handleChange函数是被缓存了的，除非count发生变化，它才会发生变化。通过demo打开控制台，发现随着count2的改变，子组件是不会重新渲染的。

#### 8.useMemo

useMemo用来缓存一个复杂的计算值。 useCallback(fn, deps) 等价于 useMemo(() => fn, deps)。如果通过一个输入得到一个值需要经过复杂的计算，那么下次同样的输入再进行一遍同样复杂的计算是没有必要的。这正是useMemo存在的意义。具体代码可以查看demo中的useMemo。

```
function Parent() {
  const [count, setCount] = useState(10);
  const [count2, setCount2] = useState(10);

  console.time('calc');
  const result = useMemo(() => computeExpensiveValue(40), [count]);
  // const result = computeExpensiveValue(count);
  console.timeEnd('calc');

  return (
    <>
      <p>result: {result}</p>
      <div>
        <input type="number" disabled value={count} />
        <button onClick={() => setCount(count + 1)}>+</button>
        <button onClick={() => setCount(count - 1)}>-</button>
        <br />
        <input type="number" disabled value={count2} />
        <button onClick={() => setCount2(count2 + 1)}>+</button>
        <button onClick={() => setCount2(count2 - 1)}>-</button>
      </div>
    </>
  );
}
```

useMemo接受一个函数，该函数涉及到复杂的计算，并接受一系列依赖项，返回一个计算后的值。如果依赖项没有变化，那么返回的值不会变化。这里useMemo依赖于count的变化。在页面中分别尝试改变count和count2的值，观察控制台输出的时间。改变count的时候，函数进行了重进计算，打印出的值比较大。改变count2的时候，直接使用缓存的值，打印出的值很小。

需要注意的是，react文档中说明，useMemo只是作为一种暗示，当依赖值变化时，并不一定能保证每一次都不计算。

#### 9.useDebugValue

通常来说你不需要它。它只会存在于自定义的hooks中用来标志一个自定义的hooks。当在chrome中打开react扩展的时候，如果一个组件使用到了自定义的hooks，并且该hooks使用到了useDebugValue，那么该组件下方会显示useDebugValue传入的参数。

```
function useUserInfo() {
  // ...

  useDebugValue('use-user-info');
  return userInfo;
}
```

## 自定义hooks

除了官方提供的hooks以外，我们也可以定义自己的hooks。编写自定义的hooks是非常简单的。你可以像和编写一个正常的组件一样，区别在于它返回的是数据（或者不返回），而不是jsx。使用自定义hooks需要遵循两点。demo中编写了一个自定义的hooks(/components/custom-hooks/use-user-info.js)。

1. 自定义hooks以use开头，驼峰命名。
2. 自定义的hooks只能被其他hooks或者函数组件使用。

```
const useUserInfo = (username = 'yuwanlin') => {
  const fetchRef = useRef(null);
  const [userInfo, setUserInfo] = useState({});
  const handleData = data => {
    setUserInfo(data);
  };
  useEffect(() => {
    const fetchData = username =>
      fetch(`${prefix}${username}`)
        .then(res => res.json())
        .then(data => {
          console.log('fetch success');
          handleData(data);
        });
    fetchRef.current = debounce(fetchData, 1000);
  }, []);

  useEffect(
    () => {
      fetchRef.current(username);
    },
    [username]
  );
  // useDebugValue('use-user-info');
  return userInfo;
};
```

打开use-user-info demo可以看到页面中出现了用户的信息。

## 如何测试hooks

[enzyme目前尚不支持测试hooks](https://github.com/airbnb/enzyme/issues/2011)。[react官方推出了测试hooks的方案](https://reactjs.org/docs/hooks-faq.html#how-to-test-components-that-use-hooks)。打开demo，在右边选项卡中。从Browser切换到Tests，可以看到通过了测试。所有位于__test__文件夹下的.test.js结尾的文件都会被当成测试文件。

## hooks缺少的部分

hooks目前不支持getSnapshotBeforeUpdate和componentDidCatch/getDerivedStateFromError生命周期，以后会加上。

## 一些常见的问题

由于每个函数就是一个组件，那么整个函数体就相当于class组件中的render函数。每次状态改变，都要重新执行函数。下面是一些常见的问题：

- 如何保存上一个状态？

在class组件中我们通过实例变量来保存上一个状态。在函数组件中，可以通过ref来取代实例变量。usePrevious的实现如下：

```
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```

- 如何实现shouldComponentUpdate？

通过React.memo。

- 在渲染的时候，由于需要创建函数，hooks是否更缓慢？

不，在现代浏览器中，与类相比，闭包的原始性能没有显著差异，除了在极端情况下。
此外，考虑到Hooks的设计在以下几个方面更有效：

1. 钩子避免了类所需的大量开销，例如在构造函数中创建类实例和绑定事件处理程序的成本。
2. 使用Hooks的惯用代码不需要深层组件树嵌套，这在使用高阶组件，渲染道具和上下文的代码库中很常见。 使用较小的组件树，React的工作量较少。

更多常见的问题，可以参照：https://reactjs.org/docs/hooks-faq.html

## hooks需要遵循的规范

- hooks只能出现在函数作用域的顶级，不能出现在条件语句、循环语句中、嵌套函数中。
- 只能从react的函数式组件以及自定义hooks中使用hooks。

代码中使用hooks的时候，最好配合eslint插件eslint-plugin-react-hooks。

```
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

## 总结

class组件存在三个问题，逻辑复用、组件庞大、难以理解的class。hooks的存在就是为了解决这三个问题的。并且在一个函数组件中，你可以多次使用相同的hooks。比如：

```
function Example() {
  const [count, setCount] = useState(0);
  const [count2, setCount2] = useState(0);

  useEffect(() => { // ... }, []);
  useEffect(() => { // ... }, []);

  return (
    // ...
  )
}
```

将不同的逻辑放到不同的hooks，逻辑更加清晰。hooks是class的另一种写法，在react16.8引入，react并不打算放弃class。
