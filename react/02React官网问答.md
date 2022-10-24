## 1. 哪个版本的 React 包含了 Hook？

从 16.8.0 开始，React 在以下模块中包含了 React Hook 的稳定实现

## 2. 有什么是 Hook 能做而 class 做不到的？

Hook 提供了强大而富有表现力的方式来在组件间复用功能。通过 「自定义 Hook」 这一节可以了解能用它做些什么。

## 3. 我应该使用 Hook，class，还是两者混用？

当你准备好了，我们鼓励你在写新组件的时候开始尝试 Hook。请确保你团队中的每个人都愿意使用它们并且熟知这份文档中的内容。我们不推荐用 Hook 重写你已有的 class，除非你本就打算重写它们。（例如：为了修复bug）。

你不能在 class 组件内部使用 Hook，但毫无疑问你可以在组件树里混合使用 class 组件和使用了 Hook 的函数组件。不论一个组件是 class 还是一个使用了 Hook 的函数，都只是这个组件的实现细节而已。长远来看，我们期望 Hook 能够成为人们编写 React 组件的主要方式。

## 4. Hook 能否覆盖 class 的所有使用场景？

我们给 Hook 设定的目标是尽早覆盖 class 的所有使用场景。目前暂时还没有对应不常用的 getSnapshotBeforeUpdate，getDerivedStateFromError 和 componentDidCatch 生命周期的 Hook 等价写法，但我们计划尽早把它们加进来。

目前 Hook 还处于早期阶段，一些第三方的库可能还暂时无法兼容 Hook。

## 5. 生命周期方法要如何对应到 Hook

constructor：函数组件不需要构造函数。你可以通过调用 useState 来初始化 state。如果计算的代价比较昂贵，你可以传一个函数给 useState。

getDerivedStateFromProps：改为 在渲染时 安排一次更新。

shouldComponentUpdate：详见 React.memo.

render：这是函数组件体本身。

componentDidMount, componentDidUpdate, componentWillUnmount：useEffect Hook 可以表达所有这些(包括 不那么 常见 的场景)的组合。

getSnapshotBeforeUpdate，componentDidCatch 以及 getDerivedStateFromError：目前还没有这些方法的 Hook 等价写法，但很快会被添加。

## 6. 有类似实例变量的东西吗

有！useRef() Hook 不仅可以用于 DOM refs。「ref」 对象是一个 c