在React中false, null, undefined, 以及true 是合法的子元素，它们并不会被渲染。

## 1. React16 和 React 15的区别

React16使用了全新的Fiber算法来分批的构建虚拟DOM对象，替代过去的一次性完成，同时生命周期上新增了getDerivedStateFromProps，getDerivedStateFromError，componentDidCatch，getSnapShortBeforeUpdate。并且不建议使用componentWillMount，componentWillReceiveProps，componentWillUpdate生命周期。

componentWillMount中的代码可以移动至constructor，componentWillReceiveProps在新的Fiber算法可能会执行多次，所以可以使用getDerivedStateFromProps这个更加接近函数式的静态方法替代，减少副作用。

## 2. React 17新特性

React17并没有增加任何的新的API，他是一个过渡版本，保留了过去所有的API使用性。同时为未来做好了铺垫。其中的几个重要的修改点有，事件委托的变更，移除事件池，修改Effect清理时机，render返回undefined报错，删除部分暴露出来的私有 API等。

时间由原来的挂载document对象改为挂载当前组件的root节点。移除事件池是不再使用包装事件对象，而拥抱原生事件对象。useEffect本身是异步执行的，但其清理工作却是同步执行的，这里同样改成了异步清理。render函数可以返回null，但如果返回undefined会报错。删除了一些私有的API，大多是当初暴露给 React Native for Web 使用的，目前 React Native for Web 新版本已经不再依赖这些API了。

总之，React17 是一个铺垫，这个版本的核心目标是让 React 能够渐进地升级，因此最大的变化是允许多版本混用，为将来新特性的平稳落地做好准备。

## 3. Hooks

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性，如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其它转化为 class。现在你可以在现有的函数组件中使用 Hook。

不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。这是因为hook是有顺序的，如果在条件中使用，顺序就会错乱。

### 1. useState

返回一个 state，以及更新 state 的函数，在初始渲染期间，返回的状态 (state) 与传入的第一个参数 (initialState) 值相同。

### 2. useEffect

Effect Hook 可以让你在函数组件中执行副作用操作，如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

与 componentDidMount 或 componentDidUpdate 不同，使用 useEffect 调度的 effect 不会阻塞浏览器更新屏幕，这让你的应用看起来响应更快。大多数情况下，effect 不需要同步地执行。在个别情况下（例如测量布局），有单独的 useLayoutEffect Hook 供你使用，其 API 与 useEffect 相同。

每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。React 会在组件卸载的时候执行清除操作。如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。与 class 组件中的 setState 方法不同，useState 不会自动合并更新对象。

### 3. 自定义hook

它就像一个正常的函数。但是它的名字应该始终以 use 开头，这样可以一眼看出其符合 Hook 的规则。

```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  // ...
  return isOnline;
}
```

比如自己编写一个useReducer。

```js
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

### 4. useContext

接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。

### 5. useReducer

useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。（如果你熟悉 Redux 的话，就已经知道它如何工作了。）

在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 useReducer 还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递 dispatch 而不是回调函数 。

### 6. useCallback

把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

### 7. useMemo

把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

记住，传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。

如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。

### 8. useRef

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

### 9. useImperativeHandle

useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。useImperativeHandle 应当与 forwardRef 一起使用。

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

### 10. useLayoutEffect

其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

### 11. useDebugValue

useDebugValue 可用于在 React 开发者工具中显示自定义 hook 的标签。

## 4. 静态方法

### 1. React.Component

React.Component 是使用 ES6 classes 方式定义 React 组件的基类。

### 2. React.PureComponent

React.PureComponent 与 React.Component 很相似。两者的区别在于 React.Component 并未实现 shouldComponentUpdate()，而 React.PureComponent 中以浅层对比 prop 和 state 的方式来实现了该函数。

### 3. React.memo

React.memo 为高阶组件。它与 React.PureComponent 非常相似，但只适用于函数组件，而不适用 class 组件。

### 4. React.createElement

创建并返回指定类型的新 React 元素。其中的类型参数既可以是标签名字符串（如 'div' 或 'span'），也可以是 React 组件 类型 （class 组件或函数组件），或是 React fragment 类型。

### 5. React.cloneElement

以 element 元素为样板克隆并返回新的 React 元素。返回元素的 props 是将新的 props 与原始元素的 props 浅层合并后的结果。新的子元素将取代现有的子元素，而来自原始元素的 key 和 ref 将被保留。

### 6. React.createFactory

返回用于生成指定类型 React 元素的函数。与 React.createElement() 相似的是，类型参数既可以是标签名字符串（像是 'div' 或 'span'），也可以是 React 组件 类型 （class 组件或函数组件），或是 React fragment 类型。

此辅助函数已废弃，建议使用 JSX 或直接调用 React.createElement() 来替代它。

### 7. React.isValidElement

验证对象是否为 React 元素，返回值为 true 或 false。

### 8. React.Children

React.Children 提供了用于处理 this.props.children 不透明数据结构的实用方法。

### 9. React.Fragment

React.Fragment 组件能够在不额外创建 DOM 元素的情况下，让 render() 方法中返回多个元素。也可以使用其简写语法 <></>

### 10. React.createRef

React.createRef 创建一个能够通过 ref 属性附加到 React 元素的 ref

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 你可以直接获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

### 11. React.forwardRef

React.forwardRef 会创建一个React组件，这个组件能够将其接受的 ref 属性转发到其组件树下的另一个组件中。

React.forwardRef 接受渲染函数作为参数。React 将使用 props 和 ref 作为参数来调用此函数。此函数应返回 React 节点。

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

### 12. React.lazy

React.lazy() 允许你定义一个动态加载的组件。这有助于缩减 bundle 的体积，并延迟加载在初次渲染时未用到的组件。

### 13. React.Suspense

React.Suspense 可以指定加载指示器（loading indicator），以防其组件树中的某些子组件尚未具备渲染条件。目前，懒加载组件是 <React.Suspense>

```js
// 该组件是动态加载的
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // 显示 <Spinner> 组件直至 OtherComponent 加载完成
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```

React.lazy() 和 <React.Suspense> 尚未在 ReactDOMServer 中支持。这是已知问题，将会在未来解决。

## 5. key

key 帮助 React 识别哪些元素改变了。元素的 key 最好是这个元素在列表中拥有的一个独一无二的字符串。当元素没有确定 id 的时候，万不得已可以使用元素索引 index 作为 key。key 只是在兄弟节点之间必须唯一。

默认情况下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。当子元素拥有 key 时，React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。相同的key会发生移动，而不是重新创建。当基于下标的组件进行重新排序时，组件 state 可能会遇到一些问题。由于组件实例是基于它们的 key 来决定是否更新以及复用，如果 key 是一个下标，那么修改顺序时会修改当前的 key，导致非受控组件的 state（比如输入框）可能相互篡改，会出现无法预期的变动。

## 6. Context

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据。Provider 接收一个 value 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 value 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 shouldComponentUpdate 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。新旧值检测来确定变化，使用了与 Object.is 相同的算法。

```js
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <ThemedButton />
      </ThemeContext.Provider>
    );
  }
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

## 7. 生命周期

```js
// 加载
constructor()

static getDerivedStateFromProps()

render()

componentDidMount()

// 更新

static getDerivedStateFromProps()

shouldComponentUpdate()

render()

getSnapshotBeforeUpdate()

componentDidUpdate()

// 卸载

componentWillUnmount()

// 错误

static getDerivedStateFromError()

componentDidCatch()
```

### 1. render

render() 方法是 class 组件中唯一必须实现的方法，会检查 this.props 和 this.state 的变化。

### 2. constructor

在 React 组件挂载之前，会调用它的构造函数。避免将 props 的值复制给 state！这是一个常见的错误。

### 3. componentDidMount

componentDidMount() 会在组件挂载后（插入 DOM 树中）立即调用。依赖于 DOM 节点的初始化应该放在这里。如需通过网络请求获取数据，此处是实例化请求的好地方。如果添加了订阅，请不要忘记在 componentWillUnmount() 里取消订阅。可以在 componentDidMount() 里直接调用 setState()。它将触发额外渲染，但此渲染会发生在浏览器更新屏幕之前。最好还是在constructor中初始化state。

### 4. componentDidUpdate

componentDidUpdate() 会在更新后会被立即调用。首次渲染不会执行此方法。可以在 componentDidUpdate() 中直接调用 setState()，但请注意它必须被包裹在一个条件语句里，正如上述的例子那样进行处理，否则会导致死循环。

### 5. componentWillUnmount

componentWillUnmount() 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 componentDidMount() 中创建的订阅等。

### 6. shouldComponentUpdate

根据 shouldComponentUpdate() 的返回值，判断 React 组件的输出是否受当前 state 或 props 更改的影响。默认行为是 state 每次发生变化组件都会重新渲染。如果 shouldComponentUpdate() 返回值为 false，则不会调用 componentDidUpdate()。首次渲染或使用 forceUpdate() 时不会调用该方法。不建议在 shouldComponentUpdate() 中进行深层比较或使用 JSON.stringify()。这样非常影响效率，且会损害性能。

### 7. static getDerivedStateFromProps

getDerivedStateFromProps 会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。

### 8. getSnapshotBeforeUpdate

getSnapshotBeforeUpdate() 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为 componentDidUpdate() 的第三个参数 “snapshot” 参数传递。应返回 snapshot 的值（或 null）。

### 9. Error boundaries

Error boundaries 仅捕获组件树中以下组件中的错误。但它本身的错误无法捕获。

static getDerivedStateFromError会在后代组件抛出错误后被调用。 它将抛出的错误作为参数，并返回一个值以更新 state。getDerivedStateFromError() 会在渲染阶段调用，因此不允许出现副作用。 如遇此类情况，请改用 componentDidCatch()。

componentDidCatch在后代组件抛出错误后被调用。 它接收两个参数：error 抛出的错误和带有 componentStack key 的对象，其中包含有关组件引发错误的栈信息。

错误边界无法捕获以下场景中产生的错误：

事件处理（了解更多）
异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
服务端渲染
它自身抛出来的错误（并非它的子组件）

自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。

### 10. UNSAFE_componentWillMount

UNSAFE_componentWillMount() 在挂载之前被调用。它在 render() 之前调用，因此在此方法中同步调用 setState() 不会触发额外渲染。通常，我们建议使用 constructor() 来初始化 state。

### 11. UNSAFE_componentWillReceiveProps

如果你需要执行副作用（例如，数据提取或动画）以响应 props 中的更改，请改用 componentDidUpdate 生命周期。如果你使用 componentWillReceiveProps 仅在 prop 更改时重新计算某些数据。在挂载过程中，React 不会针对初始 props 调用 UNSAFE_componentWillReceiveProps()。组件只会在组件的 props 更新时调用此方法。调用 this.setState() 通常不会触发 UNSAFE_componentWillReceiveProps()。

### 12. UNSAFE_componentWillUpdate

当组件收到新的 props 或 state 时，会在渲染之前调用 UNSAFE_componentWillUpdate()。使用此作为在更新发生之前执行准备更新的机会。初始渲染不会调用此方法。

此方法可以替换为 componentDidUpdate()。如果你在此方法中读取 DOM 信息（例如，为了保存滚动位置），则可以将此逻辑移至 getSnapshotBeforeUpdate() 中。

### 13. setState

setState() 将对组件 state 的更改排入队列，并通知 React 需要使用更新后的 state 重新渲染此组件及其子组件。这是用于更新用户界面以响应事件处理器和处理服务器数据的主要方式

将 setState() 视为请求而不是立即更新组件的命令。为了更好的感知性能，React 会延迟调用它，然后通过一次传递更新多个组件。React 并不会保证 state 的变更会立即生效。

除非 shouldComponentUpdate() 返回 false，否则 setState() 将始终执行重新渲染操作。

```js
this.setState((state, props) => {
  return {counter: state.counter + props.step};
});
```

### 14. forceUpdate

调用 forceUpdate() 将致使组件调用 render() 方法，此操作会跳过该组件的 shouldComponentUpdate()。但其子组件会触发正常的生命周期方法，包括 shouldComponentUpdate() 方法。如果标记发生变化，React 仍将只更新 DOM。

### 15. defaultProps

defaultProps 可以为 Class 组件添加默认 props。

### 16. displayName

displayName 字符串多用于调试消息。通常，你不需要设置它，因为它可以根据函数组件或 class 组件的名称推断出来。

## 8. ReactDOM

### 1. render

在提供的 container 里渲染一个 React 元素，并返回对该组件的引用，如果 React 元素之前已经在 container 里渲染过，这将会对其执行更新操作，并仅会在必要时改变 DOM 以映射最新的 React 元素。

### 2. hydrate

与 render() 相同，但它用于在 ReactDOMServer 渲染的容器中对 HTML 的内容进行 hydrate 操作。

### 3. unmountComponentAtNode

从 DOM 中卸载组件，会将其事件处理器（event handlers）和 state 一并清除。如果指定容器上没有对应已挂载的组件，这个函数什么也不会做。如果组件被移除将会返回 true，如果没有组件可被移除将会返回 false。

### 4. findDOMNode

如果组件已经被挂载到 DOM 上，此方法会返回浏览器中相应的原生 DOM 元素。此方法对于从 DOM 中读取值很有用，例如获取表单字段的值或者执行 DOM 检测，

### 5. createPortal

创建 portal。Portal 将提供一种将子节点渲染到 DOM 节点中的方式，该节点存在于 DOM 组件的层次结构之外。

## 9. ReactDOMServer

### 1. renderToString

将 React 元素渲染为初始 HTML。React 将返回一个 HTML 字符串。你可以使用此方法在服务端生成 HTML。

### 2. renderToStaticMarkup

此方法与 renderToString 相似，但此方法不会在 React 内部创建的额外 DOM 属性，例如 data-reactroot。

### 3. renderToNodeStream

将一个 React 元素渲染成其初始 HTML。返回一个可输出 HTML 字符串的可读流。这个 API 仅允许在服务端使用。不允许在浏览器使用。

### 4. renderToStaticNodeStream

此方法与 renderToNodeStream 相似，但此方法不会在 React 内部创建的额外 DOM 属性，例如 data-reactroot。

## 10. diff算法

在某一时间节点调用 React 的 render() 方法，会创建一棵由 React 元素组成的树。在下一次 state 或 props 更新时，相同的 render() 方法会返回一棵不同的树。React 需要基于这两棵树之间的差别来判断如何有效率的更新 UI 以保证当前 UI 与最新的树保持同步。

然而，即使在最前沿的算法中，该算法的复杂程度为 O(n 3 )，其中 n 是树中元素的数量。于是 React 在以下两个假设的基础之上提出了一套 O(n) 的启发式算法。两个不同类型的元素会产生出不同的树；开发者可以通过 key prop 来暗示哪些子元素在不同的渲染下能保持稳定；

对比两颗树时，React 首先比较两棵树的根节点。不同类型的根节点元素会有不同的形态。

比对不同类型的元素，当根节点为不同类型的元素时，React 会拆卸原有的树并且建立起新的树。

当拆卸一棵树时，对应的 DOM 节点也会被销毁。组件实例将执行 componentWillUnmount() 方法。当建立一棵新的树时，对应的 DOM 节点会被创建以及插入到 DOM 中。组件实例将执行 componentWillMount() 方法，紧接着 componentDidMount() 方法。所有跟之前的树所关联的 state 也会被销毁。

当比对两个相同类型的 React 元素时，React 会保留 DOM 节点，仅比对及更新有改变的属性。当一个组件更新时，组件实例保持不变，这样 state 在跨越不同的渲染时保持一致。React 将更新该组件实例的 props 以跟最新的元素保持一致，并且调用该实例的 componentWillReceiveProps() 和 componentWillUpdate() 方法。diff 算法将在之前的结果以及新的结果中进行递归。

在默认条件下，当递归 DOM 节点的子元素时，React 会同时遍历两个子元素的列表；当产生差异时，生成一个 mutation。

在子元素列表末尾新增元素时，更变开销比较小。在列表头部插入会很影响性能。开销会比较大。React 支持 key 属性。当子元素拥有 key 时，React 使用 key 来匹配原有树上的子元素以及最新树上的子元素。

也可以使用元素在数组中的下标作为 key。这个策略在元素不进行重新排序时比较合适，但一旦有顺序修改，diff 就会变得慢。

当基于下标的组件进行重新排序时，组件 state 可能会遇到一些问题。由于组件实例是基于它们的 key 来决定是否更新以及复用，如果 key 是一个下标，那么修改顺序时会修改当前的 key，导致非受控组件的 state（比如输入框）可能相互篡改导致无法预期的变动。

请谨记协调算法是一个实现细节。React 可以在每个 action 之后对整个应用进行重新渲染，得到的最终结果也会是一样的。在此情境下，重新渲染表示在所有组件内调用 render 方法，这不代表 React 会卸载或装载它们。React 只会基于以上提到的规则来决定如何进行差异的合并。

Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 Math.random() 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失。

## 11. 严格模式

StrictMode 是一个用来突出显示应用程序中潜在问题的工具。与 Fragment 一样，StrictMode 不会渲染任何可见的 UI。它为其后代元素触发额外的检查和警告。

StrictMode 目前有助于：识别不安全的生命周期，关于使用过时字符串 ref API 的警告，关于使用废弃的 findDOMNode 方法的警告，检测意外的副作用，检测过时的 context API。

## 12. 属性

### 1. checked

组件的 type 类型为 checkbox 或 radio 时，组件支持 checked 属性

### 2. dangerouslySetInnerHTML

dangerouslySetInnerHTML 是 React 为浏览器 DOM 提供 innerHTML 的替换方案。

### 3. htmlFor

由于 for 在 JavaScript 中是保留字，所以 React 元素中使用了 htmlFor 来代替。

### 4. onChange

onChange 事件与预期行为一致：每当表单字段变化时，该事件都会被触发。

### 5. selected

```<option> ```组件支持 selected 属性。你可以使用该属性设置组件是否被选择。这对构建受控组件很有帮助。

### 6. suppressContentEditableWarning

当拥有子节点的元素被标记为 contentEditable 时，React 会发出一个警告，因为这不会生效。该属性将禁止此警告。

### 7. suppressHydrationWarning

如果你使用 React 服务端渲染，通常会在当服务端与客户端渲染不同的内容时发出警告。
