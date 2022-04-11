React中数据流向是单向的，而且总是自上而下传递的，可以通过props将数据从父组件传递给子组件，但是假设需要将组件树最底层的Banner节点的数据传递给最顶层的Index，这个时候组件之间该如何通信呢。

遵循React的单向数据传递原则我们是没有办法直接传递数据的不过我们可以通过函数回调的方式，通过调用父组件的函数一层一层的向上传递。也就是Banner调用回调将数据传给Main，Main再通过回调将数据传给Index。

在大型的网站中类似这样需要共享数据的情况非常常见，如果通过回调函数这样来一层一层传递你会发现整个网站的代码会变得非常恶心。基本上你的代码就是无法维护的状态。而且这样处理数据的开销是非常巨大的。一个不小心很有可能陷入无限死循环中。

所以当我们的网站复杂到一定程度的时候我们就需要设计模式了，可能之前你已经知道MVC, MVVM, MV*。但是针对React我们还可以使用一种更加符合React设计思想的架构模式，Redux。

Redux是一种设计模式同时也是一种项目架构方案，他不依赖任何库或者任何框架，他不仅可以在React中使用甚至在Angular和Vue中也可以使用。

使用Redux架构来说所有的组件基本不会互相通信了，数据放在一个叫做store的数据仓库中存储。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/284e30439e644619832e0736bbc3812a~tplv-k3u1fbpfcp-watermark.image)

通过使用Redux我们可以剥离出组件中的数据(state),将所有数据统一存放在Redux数据(store)仓库中,如果组件中哪一个组件需要使用到数据，这个组件可以去数据仓库中自行认领有个高大上的叫法是订阅。如果组件中对store中的数据进行了更新那么store会向订阅了这个数据的所有组件推送最新的数据，这就是Redux的原理。

Redux就是数据仓库，他把数据统一保存起来，在隔离的数据和UI的同时还处理了他们之间的关系。

使用Redux的目的是让状态state的变化可控可预测。虽然从原理来看Redux似乎挺简单的但是想要了解他的工作流程就比较麻烦了。

这主要是因为他的数据流动方式不是特别直观，有点类似事件驱动的方式，我们知道事件驱动开发最困难的地方是在调试。Redux中使用了很多晦涩难懂的专业术语比如Action，Reducer，Dispatch等，了解这些名词之前我们很难把握Redux的方向。还有就是Redux的文档并不亲民，到处都是新概念，比如说纯函数，flux，observable，immutable这些概念张口就来完全不去考虑别人是否可以看懂。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4eff3171612a43bf9f57478c9646b91c~tplv-k3u1fbpfcp-watermark.image)

一般来说使用Redux都会创建一个用于存放数据的Store，在这个Store中有若干个Reducer，然后我们需要使用React组件来渲染UI，除此之外还会有若干个和Reducer对应的Action指令。

Store中的Reducer组合在一起就形成了项目中的数据仓库。Redux称之为State也就是数据。React组件通过订阅(subscribe )Store来获得数据，然后使用数据来渲染UI,UI通过显示器显示给用户，用户通过鼠标和键盘与组件进行交互，在交互中不可避免需要改变数据，在React中数据的流动是单向的，所以对数据来说React组件只有读取权限，没有书写权限UI组件不可以直接访问Store修改数据。

所以UI必须向Store发送Action指令，来让Store自己修改自己，这个指令的分发过程就叫做dispatch。Action指令到达store之后可能会经过若干个middleware中间件进行数据的预处理，对于数据的异步处理也是在这里进行的，预处理过后数据就会连同action一起传递给reducer，reducer会按照Action中描述的指令来更新数据state，当state更新好以后Store就会把数据推送给订阅了自己的组件，组件会根据新的数据重新渲染UI, 用户就能看到变化了。可以看到在实际工作中Redux架构还是相对复杂的。

上面的描述还是比较复杂的，不过不要慌，下面我们来简化一下这张图，只保留几个主要部件，通过学习简化的流程来了解Redux。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e392031194240a8b3f85b836f73ed72~tplv-k3u1fbpfcp-watermark.image)

简化后的六层我们只保留Reducer，Store，React组件，Actions这四个部分。为了更加清晰我们这里将Reducer从Store中移了出来，实际上他们是一体的。

Store中保存的是全局数据，对于Redux项目来说有且只有一个Store，我们可以把它看做一个带有推送功能的数据仓库。我们可以借用微信的朋友圈来理解这个概念。比如你加了某个人的好友，只要这个人一发朋友圈他的状态就会马上推送到你。加好友就是数据订阅，发朋友圈就是数据推送。

Reducer是帮助Store处理数据的方法，他是一个方法是一个过程是一个函数不是一个具体存在的对象，Reducer可以帮助Store初始化数据，修改数据，删除数据，你可能会好奇我们为什么要使用Reducer这么麻烦的方式来处理数据而不是直接在Store中进行修改，其实原因也很简单。比如你看到你朋友的朋友圈有错别字，你是没办法直接修改它的朋友圈状态的。

任何UI级别的组件都没有权限修改Store中的数据，根据数据单向流动的原则他们是只读不能写的，你只能给他打电话或者发短通知他让他来修改，他修改后会从新推送给你。给他打电话或者发短信通知他就是Action，给他打电话或者发短信通知他的这个过程就是dispatch Action消息的分发。他自己修改朋友圈的过程就是reducer。

最后他修改好之后微信会从新将消息通送给你，这就是订阅和推送。

所以Store就是Redux中具有推送功能的数据仓库，Reducer是Store处理数据的方法可以帮助Store实现数据的初始化，修改或者删除，Actions就是数据更新的指令，他会告诉Reducer如何去处理数据所以Redux的流程其实很清晰。

首先创建数据仓库Store，Reducer会同时初始化数据state。React Component会订阅Store，Store中的数据就会被推送过来，然后渲染UI.

如果组件需要更改数据他会发送一个Action，这个过程就叫做dispatch。Action会以事件驱动的方式被Store所截获，Store会将自己当前的数据以及指令传递给Reducer，由Reducer去更新数据。Reducer更新完成以后就会向Store输出一个新的state，Store取到新的state之后就会向订阅了自己的React组件推送这个新的数据。然后重新再次渲染UI。这就是一个完整的Redux工作流程。

Redux是一种设计模式同时也是一种项目架构方案，他不依赖任何库或者任何框架，只是大家习惯于将Redux和React放在一起使用。这里我们介绍一下Redux的使用，为了避免混淆我们不使用任何框架。

首先你可以通过npm在项目中安装redux插件，前面说过Store就是保存数据的地方，整个应用只能有一个Store, Redux提供createStore这个函数，用来生成Store。

```js
import { createStore } from 'redux'
const store = createStore(fn);
```

这里createStore需要接收一个函数，这个函数就是用来处理action操作的也就是我们之前说的Reducer，所以他需要接收action参数，因为他是帮助Store处理数据的，所以也需要接收源数据，返回值是更新后的数据。

```js
const fn = (state = 0, action) => {
    switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

在需要使用数据的位置我们可以通过getState来获取数据，通过subscribe订阅来监听数据的变化，因为Redux是一种发布订阅模式，只有监听才会获取到。

```js
store.subscribe(() => {
    const state = store.getState();
}))
```

需要更数据时，需要使用dispatch配合action来分发。我们约定action需要是一个拥有type属性的对象，type来表示要操作的类型，如果传递参数我们一般将参数放在payload属性中。

```js
const action = { 
    type: 'INCREMENT',
    payload: '参数',
};
store.dispatch(action);
```

这样我们当调用store.dispatch时，Redux会将action传递给Reducer，Reducer通过自身的逻辑处理返回新的state，然后Redux记录这个新的state并且推送消息给订阅了自己的组件。也就是会触发subscribe中传入的函数。函数中可以通过store.getState()获得新的state值，完成页面更新。

假设我们页面中有一个button按钮和一个div元素，这个元素用来展示一个数字，初始值为0，当我们点击button按钮的时候让div中显示的数字增加。

```html
<div id="count"></div>
<button id="button">按钮</button>
```

js代码如下, 我们首先定义reducer，在里面判断如果type为INCREMENT就让state+1，然后通过createStore创建store传入处理函数reducer。

接着订阅state，当state变更时获取页面div元素更新div的内容为state的值。

最后点击按钮的时候我们通过dispatch来分发action。

```js
var reducer = (state = 0, action) => {
    switch (action.type) {
    case 'INCREMENT':
      return state + 1
    default:
      return state
  }
}

var store = createStore(reducer);

store.subscribe(() => {
    document.querySelector('#count').innerHTML = store.getState();
});

document.querySelector('#button').addEventListener('click', function () {
    store.dispatch({type: 'INCREMENT'});
});
```

这就是Redux的一个基本使用过程，你懂了么？

那具体什么时候需要使用到Redux呢？

1. 组件需要共享数据或者共享状态(state)的时候；

2. 某一个组件在任何地方都需要被随时访问的时候。

3. 某一个组件需要改变另一个组件状态的时候。

4. 网站支持国际化语言切换，登录数据共享的情况下。

满足上面一种或几种情况建议使用redux，如果你还在考虑项目要不要使用redux我给的建议就是不要。技术是为了服务业务。为了避免设计的头重脚轻，建议只有在需要的时候才引入新概念，切忌为了使用而使用。
