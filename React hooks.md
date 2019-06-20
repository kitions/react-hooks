---
title:  初识React-hooks  #文章页面上的显示名称，一般是中文
categories: 默认分类 #分类
date: 2019-06-20 11:53:52
tags: [React] #文章标签，可空，多标签请用格式，注意:后面有个空格
description: Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性（生命周期等特性）。带来的好处不仅是 “更 FP，更新粒度更细，代码更清晰”
---

# React hooks

*Hook* 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性（生命周期等特性）。



## 0. 背景

 长期以来很多人会把 `Stateless Component` 和 `Functional Component` 混为一谈；

 Hooks 的出现本质是把这种**面向生命周期编程**变成了**面向业务逻辑编程**，你不用再去关心本不该关心的生命周期，写法上带来的优化只是顺带的。



## 1. 优势

​	 带来的好处不仅是 “更 FP，更新粒度更细，代码更清晰”

官方：

- **完全可选的。** 你无需重写任何已有代码就可以在一些组件中尝试 Hook。但是如果你不想，你不必现在就去学习或使用 Hook。
- **100% 向后兼容的。** Hook 不包含任何破坏性改动。
- **现在可用。**  v16.8.0以后版本都可以使用。
- **渐进策略**。 Hook 和现有代码可以同时工作，你可以渐进式地使用他们。

非官方：
- 更容易将组件的 UI 与状态分离, 状态与 UI 的界限会越来越清晰。
- 多个状态不会产生嵌套，写法还是平铺的
- Hooks 可以引用其他 Hooks。

```jsx
class Example extends React.Component{
  constructor(props){
    super(props)
    this.state = {
      count : 0
    }
  }
  return (
    <div>
      <p>You clicked {this.state.count} times</p>
      <button onClick={() => this.setState({count:this.state.count + 1})}>
        Click me
      </button>
    </div>
  );
}
```

```jsx
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 “count” 的 state 变量。
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```





可以看到，`Example`变成了一个函数，但这个函数却有自己的状态（count），同时它还可以更新自己的状态（setCount）。这个函数之所以这么了不得，就是因为它注入了一个hook--`useState`，就是这个hook让我们的函数变成了一个有状态的函数。




## 2. 动机

### （1）有状态的组件之间复用状态逻辑很难   

问题：React 需要为共享状态逻辑提供更好的原生途径。

解决：Hook 使你在无需修改组件结构的情况下复用状态逻辑



我们都知道react的**核心思想**就是，将一个页面拆成一堆独立的，可复用的组件，并且用自上而下的单向数据流的形式将这些组件串联起来。但假如你在大型的工作项目中用react，你会发现你的项目中实际上很多react组件冗长且难以复用。尤其是那些写成class的组件，它们本身包含了状态（state），所以复用这类组件就变得很麻烦。



之前，官方推荐怎么解决这个问题呢？答案是：[渲染属性（Render Props）](https://reactjs.org/docs/render-props.html)和[高阶组件（Higher-Order Components）](https://segmentfault.com/img/bVbjfuz)。我们可以稍微跑下题简单看一下这两种模式。



#### - 渲染属性（Render Props）

渲染属性指的是使用一个值为函数的prop来传递需要动态渲染的nodes或组件。如下面的代码可以看到我们的`DataProvider`组件包含了所有跟状态相关的代码，而`Cat`组件则可以是一个单纯的展示型组件，这样一来`DataProvider`就可以单独复用了。

```tsx
import Cat from 'components/cat'
class DataProvider extends React.Component {
  constructor(props) {
    super(props);
    this.state = { target: 'Zac' };
  }

  render() {
    return (
      <div>
        {this.props.render(this.state)}
      </div>
    )
  }
}

<DataProvider render={data => (
  <Cat target={data.target} />
)}/>

// 上下等同
<DataProvider>
  {data => (
    <Cat target={data.target} />
  )}
</DataProvider>
```

#### - 高阶组件

```tsx
const withUser = WrappedComponent => {
  const user = sessionStorage.getItem("user");
  return props => <WrappedComponent user={user} {...props} />;
};

const UserPage = props => (
  <div class="user-container">
    <p>My name is {props.user}!</p>
  </div>
);

export default withUser(UserPage);
```

以上这两种模式看上去都挺不错的，很多库也运用了这种模式，但我们仔细看这两种模式，会发现它们会增加我们代码的层级关系，这时候再回过头看hooks例子，是不是简洁多了，没有多余的层级嵌套

![img](https://img.alicdn.com/tfs/TB1oipbryrpK1RjSZFhXXXSdXXa-2048-860.jpg)



### （2）复杂组件变得难以理解

问题：组件起初很简单，但是逐渐会被状态逻辑和副作用充斥。相互关联且需要对照修改的代码被进行了拆分，而完全不相关的代码却在同一个方法中组合在一起

解决：Hook 将组件中相互关联的部分拆分成更小的函数（比如设置订阅或请求数据）

### （3）难以理解的 class

问题：js的`this` 的工作方式；绑定事件处理器；对于函数组件与 class 组件的差异也存在分歧

解决：Hook 使你在非 class 的情况下可以使用更多的 React 特性，Hook 则拥抱了函数

我们经常在写一个组件的时候，把组件写成无状态组件的形式，这样更方便复用，独立厕所，然而很多时候，用SFC 写了一个简洁完美的无状态组件，后来因为需求变动，必须得有状态，又得很麻烦的改成class组件。就很烦，有了hook，就可以避免这样的问题

### （4）生命周期钩子函数里的逻辑太乱！

我们通常希望一个函数只做一件事情，但我们的生命周期钩子函数里通常同时做了很多事情。比如我们需要在`componentDidMount`中发起ajax请求获取数据，绑定一些事件监听等等。同时，有时候我们还需要在`componentDidUpdate`做一遍同样的事情。当项目变复杂后，这一块的代码也变得不那么直观。



## 3. useState

```jsx
import { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);
```

`useState`是react自带的一个hook函数，它的作用就是用来声明状态变量。`useState`这个函数接收的参数是我们的状态初始值（initial state），它返回了一个数组，这个数组的第`[0]`项是当前当前的状态值，第`[1]`项是可以改变状态值的方法函数。

### 读取状态值

```tsx
<p>You clicked {count} times</p>
```

### 更新状态

```jsx
  <button onClick={() => setCount(count + 1)}>
    Click me
  </button>
```
### 多个状态值

```jsx
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```

从ExampleWithManyStates函数我们可以看到，useState无论调用多少次，相互之间是独立的





（[不推荐](https://zh-hans.reactjs.org/docs/hooks-intro.html#gradual-adoption-strategy)把你已有的组件全部重写，但是你可以在新组件里开始使用 Hook。）

```react
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```





## 4. useEffect

可以把 `useEffect` Hook 看做 `componentDidMount`，`componentDidUpdate` 和 `componentWillUnmount` 这三个函数的组合。

默认情况下，它在第一次渲染之后*和* 每次更新之后都会执行。

### Class

```react
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```



### hook

```react
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 类似于componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 更新文档的标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```



effect 有可选的清除机制。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。

### React 何时清除 effect？

 React 会在组件卸载的时候执行清除操作。effect 在每次渲染的时候都会执行。这就是为什么 React *会*在执行当前 effect 之前对上一个 effect 进行清除。稍后[为什么这将助于避免 bug](https://zh-hans.reactjs.org/docs/hooks-effect.html#explanation-why-effects-run-on-each-update)以及[如何在遇到性能问题时跳过此行为](https://zh-hans.reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects)。

> Tips
>
> 与 `componentDidMount` 或 `componentDidUpdate` 不同，使用 `useEffect` 调度的 effect 不会阻塞浏览器更新屏幕，这让你的应用看起来响应更快。大多数情况下，effect 不需要同步地执行。在个别情况下（例如测量布局），有单独的 [`useLayoutEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect) Hook 供你使用，其 API 与 `useEffect` 相同。



### 通过跳过 Effect 进行性能优化

在某些情况下，每次渲染后都执行清理或者执行 effect 可能会导致性能问题。在 class 组件中，我们可以通过在 `componentDidUpdate` 中添加对 `prevProps` 或 `prevState` 的比较逻辑解决：

```jsx
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

​		这是很常见的需求，所以它被内置到了 `useEffect` 的 Hook API 中。如果某些特定值在两次重渲染之间没有发生变化，你可以通知 React **跳过 **对 effect 的调用，只要传递数组作为 `useEffect` 的第二个可选参数即可：

```jsx
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```

​		这个参数是什么作用呢？如果 `count`的值是 `5`，而且我们的组件重渲染的时候 `count` 还是等于 `5`，React 将对前一次渲染的 `[5]`和后一次渲染的 `[5]` 进行比较。因为数组中的所有元素都是相等的(`5 === 5`)，React 会跳过这个 effect，这就实现了性能的优化。



> 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行







## 5. 还有哪些自带的Effect Hooks?

除了上面介绍的useState和useEffect，react还给我们提供来很多有用的hooks：

useContext
useReducer
useCallback
useMemo
useRef
useImperativeMethods
useMutationEffect
useLayoutEffect

我不再一一介绍，大家自行去查阅官方文档。







## 6. 自定义hook

当我们想在两个函数之间共享逻辑时，我们会把它提取到第三个函数中。而组件和 Hook 都是函数，所以也同样适用这种方式。

**自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook。** 

> Hook 函数必须以 "use" 命名开头，这种声明目前是通过很弱的 `use` 前缀标识的（但是设计上会简洁很多），为了不弄错每个盒子和状态的对应关系，书写的时候 Hooks 需要 `use` 开头且放在顶层作用域，即不可以包裹 `if/switch/when/try` 等。引入了官方的 [eslint-plugin-react-hooks](https://link.juejin.im/?target=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Feslint-plugin-react-hooks)  就不用担心会弄错了。



为什么不能用 condition 包裹 useHook 语句，详情可以见 [官方文档](https://link.juejin.im/?target=https%3A%2F%2Freactjs.org%2Fdocs%2Fhooks-rules.html%23explanation)，这里简单介绍一下。

React Hooks 并不是通过 Proxy 或者 getters 实现的（具体可以看这篇文章 [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)），而是通过数组实现的，每次 `useState` 都会改变下标，如果 `useState` 被包裹在 condition 中，那每次执行的下标就可能对不上，导致 `useState` 导出的 `setter` 更新错数据。



## 7. 总结

- Hooks本质是把**面向生命周期程式设计**变成了**面向业务逻辑程式设计**；
- Hooks 是React 的未来，但还是无法完全替代原始的Class。





## Example 1:

我们先假想一个常见的需求，一个 Modal 里需要展示一些信息，这些信息需要通过 API 获取且跟 Modal 强业务相关, Modal 打开的时候才进行数据获取：

[代码](https://codepen.io/int64ago/pen/qQoJOX?editors=0010)

```tsx
class RandomUserModal extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: {},
      loading: false,
    };
    this.fetchData = this.fetchData.bind(this);
  }

  componentDidMount() {
    if (this.props.visible) {
      this.fetchData();
    }
  }

  componentDidUpdate(prevProps) {
    if (!prevProps.visible && this.props.visible) {
      this.fetchData();
    }
  }

  fetchData() {
    this.setState({ loading: true });
    fetch('https://randomuser.me/api/')
      .then(res => res.json())
      .then(json => this.setState({
        user: json.results[0],
        loading: false,
      }));
  }

  render() {
    const user = this.state.user;
    return (
      <ReactModal
        isOpen={this.props.visible}
      >
        <button onClick={this.props.handleCloseModal}>Close Modal</button>
        {this.state.loading ?
          <div>loading...</div>
          :
          <ul>
            <li>Name: {`${(user.name || {}).first} ${(user.name || {}).last}`}</li>
            <li>Gender: {user.gender}</li>
            <li>Phone: {user.phone}</li>
          </ul>
        }
      </ReactModal>
    )
  }
}
```

为了实现在 Modal 打开的时候才进行数据获取，我们需要同时在 `componentDidMount` 和 `componentDidUpdate` 两个生命周期里实现数据获取的逻辑，而且 `constructor` 里的一些初始化操作也少不了。

其实我们的要求很简单：在合适的时候通过 API 获取新的信息，这就是我们抽象出来的一个**业务逻辑**，为了这个业务逻辑能在 React 里正确工作，我们需要将其**按照 React 组件生命周期进行拆解**。这种拆解除了**代码冗余**，还**很难复用**。



[hooks的改造后](https://codepen.io/int64ago/pen/eQMLNX?editors=0010)：

```tsx
function RandomUserModal(props) {
  const [user, setUser] = React.useState({});
  const [loading, setLoading] = React.useState(false);

  React.useEffect(() => {
    if (!props.visible) return;
    setLoading(true);
    fetch('https://randomuser.me/api/').then(res => res.json()).then(json => {
      setUser(json.results[0]);
      setLoading(false);
    });
  }, [props.visible]);
  
  return (
    // View 部分几乎与上面相同
  );
}
```

很明显地可以看到我们把 Class 形式变成了 Function 形式，使用了两个 State Hook 进行数据管理（类比 `constructor`），之前 `componentDidMount` 和 `componentDidUpdate` 两个生命周期里干的事我们直接在一个 Effect Hook 里做了。做了这些，最大的优势是**代码精简**，业务逻辑变的紧凑，代码行数也从 50+ 行减少到 30+ 行。

Hooks 的强大之处还不仅仅是这个，最重要的是这些业务逻辑可以随意地的的抽离出去，跟普通的函数没什么区别（仅仅是看起来没区别），于是就变成了可以**复用**的自定义 Hook。具体可以看下面的进一步改造：



```jsx
// 自定义 Hook
function useFetchUser(visible) {
  const [user, setUser] = React.useState({});
  const [loading, setLoading] = React.useState(false);
  
  React.useEffect(() => {
    if (!visible) return;
    setLoading(true);
    fetch('https://randomuser.me/api/').then(res => res.json()).then(json => {
      setUser(json.results[0]);
      setLoading(false);
    });
  }, [visible]);
  return { user, loading };
}

function RandomUserModal(props) {
  const { user, loading } = useFetchUser(props.visible);
  
  return (
    // 与上面相同
  );
}
```

这里的 `useFetchUser` 为自定义 Hook，它的地位跟自带的 `useState` 等比也没什么区别，你可以在其它组件里使用，甚至在这个组件里使用两次，它们会天然地隔离开。



## Example 2:

[实例二](https://codesandbox.io/s/jvvkoo8pq3)







参考：

 [React Hooks 深入不浅出](https://segmentfault.com/a/1190000017182184)

 [30分钟精通React今年最劲爆的新特性——React Hooks](https://segmentfault.com/a/1190000016950339)

 [React hooks: not magic, just arrays ](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)

 [Making Sense of React Hooks ](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889)

 [A Complete Guide to useEffect  ](https://overreacted.io/a-complete-guide-to-useeffect/)  推荐 Dan 的这篇文章

 [Under the hood of React’s hooks system](https://medium.com/the-guild/under-the-hood-of-reacts-hooks-system-eb59638c9dba)  原理  [中文版](https://juejin.im/post/5c99a75af265da60ef635898)