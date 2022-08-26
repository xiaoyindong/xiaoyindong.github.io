## JSX是什么

JSX是看起来像HTML实际上是JavaScript，在React代码执行之前Babel会将JSX编译为React API。也就是一个JavaScript对象。

```jsx
// 编译前
<div className="content">
    <h3>Hello React</h3>
    <p>React is great</p>
</div>
// 编译后
React.createElement(
    'div',
    {
        className: 'content'
    },
    React.createElement('h3', null, 'Hello World'),
    React.createElement('p', null, 'React is greate')
)
```

React.createElement代表一个具体的节点元素，第一个参数是节点名称，第二个是节点属性，后面的参数是子节点列表。

jsx在运行时会被Babel转换为React.createElement对象，React.createElement会被React转换成虚拟DOM对象，虚拟DOM对象又会被React转换成真实DOM对象。JSX语法的出现就是为了让React开发人员编写用户界面代码更加轻松。

## 虚拟DOM

在React中，每个DOM对象都有一个对应的虚拟DOM对象，其实就是使用JavaScript对象来描述DOM对象信息，比如DOM对象的类型，属性，子元素。

```jsx
// 编译前
<div className="content">
    <h3>Hello React</h3>
</div>
// 编译后
{
    type: "div",
    props: { className: "content"},
    children: [
        {
            type: "h3",
            props: null,
            children: [
                {
                    type: "text",
                    props: {
                        textContent: "Hello React"
                    }
                }
            ]
        }
    ]
}
```

React采用最小化的DOM操作来提升DOM操作的优势，只更新需要更新的，在React第一次创建DOM对象的时候会为每一个DOM对象创建虚拟的DOM对象，在DOM对象发生更新之前React会更新所有的虚拟DOM对象, 然后将更新前的虚拟DOM和更新后的虚拟DOM进行对比，找到变更的DOM对象，只将发生变化的DOM更新到页面中从而提升了js操作DOM的性能。

虽然在操作真实DOM之前进行的虚拟DOM更新和对比的操作，但是由于JS操作自有对象效率是很高的，成本几乎可以忽略不计的。

## Fiber

React16之前的版本比对更新虚拟DOM的过程是采用循环递归方式来实现的，这种比对方式有一个问题，就是一旦任务开始进行就无法中断，如果应用中数组数量庞大，主线程被长期占用，直到整颗虚拟DOM树比对更新完成之后主线程才被释放，主线程才能执行其他任务，这就会导致一些用户交互或动画等任务无法立即得到执行，页面就会产生卡顿，非常的影响用户体验。

Fiber是一种DOM比对新算法，采用的利用浏览器空余时间来执行任务，拒绝长时间占用主线程。放弃递归只采用循环，将大的任务拆分成一个个的小任务来执行。以前整颗DOM树的比对是一个任务，现在改成每个节点的比对是一个任务。

在Fiber方案中，为了实现任务的终止再结束，DOM比对算法被分成了两个部分，第一部分是虚拟DOM的比对，第二个是真实DOM的更新。虚拟DOM的比对是可以终止的，真实DOM的更新是不能终止的。

React编写页面使用的是jsx语法，babel会将jsx转换为React.createElement方法的调用，返回虚拟DOM对象，然后就去执行第一个阶段也就是构建Fiber对象。我们通过循环在虚拟DOM中找到每个内部的对象，为每一个虚拟DOM对象构建Fiber对象，Fiber和虚拟DOM对象类似。

```js
{
    type: "节点类型",
    props: "节点属性",
    stateNode: "节点DOM对象 或 组件实例对象",
    tag: "节点标记",
    effects: ['存储需要更改的fiber对象'],
    effectTag: "当前Fiber要被执行的操作",
    parent: "当前Fiber的父级Fiber",
    child: "当前Fiber的子级Fiber",
    sibling: "当前Fiber的兄弟Fiber",
    alternate: "Fiber备份用于比对时使用"
}
```

当所有的Fiber节点生成之后存储在数组中，就可以执行第二阶段的操作循环Fiber数组，在循环的过程中根据effectTag来确定当前节点要做的操作将它应用在真实的DOM当中。

总结来说就是如果是初始渲染首先需要构建Fiber对象再将Fiber对象存储在数组当中，然后将Fiber要做的操作应用在真实的DOM当中，如果是状态更新就重新构建所有的Fiber对象再获取到旧的Fiber对象进行比对，然后形成Fiber数组，然后再将Fiber应用到真实DOM当中。

在第二阶段的所有的Fiber对象都在同一个数组中他们的关系完全被抹平了，页面中的每一个DOM元素都是一个子元素，所以Fiber对象中还存储了当前节点的父级，子级，兄弟级。

- requestIdleCallback

requestIdleCallback是React-Fiber中用到的核心API，他的作用是利用浏览器的空余时间执行任务，如果有更高优先级的任务要执行时，当前执行的任务可以被终止，优先执行高级别任务。

requestIdleCallback这是浏览器内置的方法，可以直接使用。接收一个回调函数，函数中接收一个对象作为参数，对象中存在timeRemaining方法返回空余时间的毫秒。

```js
requestIdleCallback(function(deadline) {
    // todo
    // deadline.timeRemaining() 获取浏览器的空余时间
})
```

## diff 算法

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

## useState实现

先声明一下useState函数，返