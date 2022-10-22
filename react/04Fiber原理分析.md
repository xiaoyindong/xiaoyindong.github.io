React16之前的版本比对更新虚拟DOM的过程是采用循环递归方式来实现的，这种比对方式有一个问题，就是一旦任务开始进行就无法中断，如果应用中数组数量庞大，主线程被长期占用，直到整颗虚拟DOM树比对更新完成之后主线程才被释放，主线程才能执行其他任务，这就会导致一些用户交互或动画等任务无法立即得到执行，页面就会产生卡顿，非常的影响用户体验。

Fiber是一种DOM比对新算法，以前的算法叫做stack，正是由于Stack算法存在上面的问题，React官方才重写了比对算法。Fiber采用的利用浏览器空余时间来执行任务，拒绝长时间占用主线程。放弃递归只采用循环，将大的任务拆分成一个个的小任务来执行。以前整颗DOM树的比对是一个任务，现在改成每个节点的比对是一个任务。

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

## 1. requestIdleCallback

requestIdleCallback是React-Fiber中用到的核心API，他的作用是利用浏览器的空余时间执行任务，如果有更高优先级的任务要执行时，当前执行的任务可以被终止，优先执行高级别任务。

requestIdleCallback这是浏览器内置的方法，可以直接使用。接收一个回调函数，函数中接收一个对象作为参数，对象中存在timeRemaining方法返回空余时间的毫秒。

```js
requestIdleCallback(function(deadline) {
    // todo
    // deadline.timeRemaining() 获取浏览器的空余时间
})
```

页面是按帧绘制出来的，当每秒绘制帧数达到60时页面是流畅的，小于这个值用户会觉得卡顿。也就是每16ms绘制一帧，如果每一帧执行的时间小于16ms就说明浏览器有空余时间，如果任务在剩余的时间内没有完成则会停止任务执行，继续优先执行主任务，也就是说requestIdleCallback总是利用浏览器的空余时间执行任务。

```js
function calc(deadline) {
    while (number > 0 && deadline.timeRemaining() > 1) {
        value = Math.random() > 0.5 ? value + Math.random() : value + Math.random()
        number--;
    }
    requestIdleCallback(calc);
}
```

## 2. 创建任务队列

下面我们就要开始去实现Fiber算法，在src/index.js中加入下面这段代码。

```js
import React, { render } from './react';
const jsx = <div>
    <p>Hello Fiber</p>
</div>

const root = document.getElementById('root');
console.log(jsx);
render(jsx, root)
```

可以看到控制台输出了一个对象，可以看到第一标签是div，子元素是p标签，p标签的子元素是text文本元素。

```json
{
    "type": "div",
    "props": {
        "children": [
            {
                "type": "p",
                "props": {
                    "children": [
                        {
                            "type": "text",
                            "props": {
                                "children": [],
                                "textContent": "Hello Fiber"
                            }
                        }
                    ]
                }
            }
        ]
    }
}
```


定义render方法，这个函数接收两个参数，虚拟DOM和要渲染的容器。在这个函数中需要做两件事，向任务队列中添加任务，和指定在浏览器空闲时执行任务。在render方法中向队列中添加任务，一个任务就是一个对象，这个对象有父级和子级两个属性。

```js
import { createTaskQueue } from '../Misc';
const taskQueue = createTaskQueue();

export const render = (element, dom) => {
    // 1. 向任务队列中添加任务
    taskQueue.push({
        dom, // 父级
        props: {
            children: element // 子级
        }
    })
    // 2. 指定在浏览器空闲时执行任务
}
```

定义一个队列createTaskQueue，可以向队列中添加内容和读取内容。

```js
const createTaskQueue = () => {
    const taskQueue = [];
    return {
        push: item => { // 向任务队列中添加任务
            return taskQueue.push(item);
        },
        pop: () => { // 从任务队列中获取任务
            return taskQueue.shift();
        },
        isEmpty: () => { // 判断是否存在任务
            taskQueue.length === 0
        }
    }
}

export default createTaskQueue;
```

## 3. 实现任务的调度逻辑

在render方法中调用requestIdleCallback在浏览器空闲的时候去执行任务。

```js
const render = (element, dom) => {
    // 1. 向任务队列中添加任务
    taskQueue.push({
        dom, // 父级
        props: {
            children: element // 子级
        }
    })
    // 2. 指定在浏览器空闲时执行任务
    requestIdleCallback(performTask)
}
```

requestIdleCallback被调用的时候需要传递一个函数作为参数，当浏览器空闲的时候就会去调用这个函数。这个方法只负责调度任务并不执行任务。

```js
const performTask = deadline => {
    workLoop(deadline)
}
```

定义workLoop方法判断任务是否存在。定义subTask来存储当前的任务，默认值为空，在workLoop中判断subTask是否存在。如果不存在调用getFirstTask方法获取一个任务。

```js
let subTask = null;

const getFirstTask = () => {
    
}

const workLoop = deadline => {
     if (!subTask) {
        subTask = getFirstTask()
     }
}
```

如果任务存在就执行任务。循环中判断subTask存在并且浏览器的空余时间大于1ms再执行任务。执行任务的代码放在executeTask函数中来单独处理。executeTask接收的对象其实就是一个Fiber。executeTask执行结束之后需要返回一个新的任务。

```js

const executeTask = fiber => {

}

const workLoop = deadline => {
    if (!subTask) {
        subTask = getFirstTask()
    }

    while (subTask && deadline,timeRemaining() > 1) {
        subTask = executeTask(subTask);
    }
}
```

requestIdleCallback是会被打断的，如果有更优先的任务执行这里就断掉了，这个方法就会退出，也就是执行到performTask中，所以我们这里要去判断一下subTask是否有值，如果有值就是任务没有执行完，同时我们也要判断任务队列中是否有值。

```js
const performTask = deadline => {
    workLoop(deadline);
    if (subTask || !taskQueue.isEmpty()) {
        requestIdleCallback(performTask)
    }
}
```

## 4. 构建Fiber对象

在任务执行之前首先需要明确要执行的是什么任务，这个任务应该怎样被执行。

要执行的任务是根据虚拟DOM独享为每一个节点构建Fiber对象。以下面的对象为例。首先先构建最外层的div对象，然后再构建里面的两个div，当这三个节点构建完成之后就要去设定他们之间的对应关系。

对于parent这个div来说，只有第一个div是他的子节点，剩余的都是child这个div的兄弟节点。当他们三个的关系建立完毕之后，再去找父级的第一个子级节点child，看这个节点是否有子级，这里有个p。当p构建完毕，这条链路就构建完了。既没有子级也没有同级。这个时候继续向父级查找，看父级是否有兄弟节点，如果存在兄弟节点，判断是否已经构建如果已经构建完了再检查这个节点的子节点。如果没有构建就构建之后再检查子节点。

```html
<div class="parent">
    <div class="child">
        <p></p>
    </div>
    <div class="sibling">
        <p></p>
    </div>
</div>
```

在这个jsx中root是父节点，div是子节点，这里我们已经通过render方法传递进去了。

```jsx
import React, { render } from './react';
const jsx = <div>
    <p>Hello Fiber</p>
</div>

const root = document.getElementById('root');

render(jsx, root)
```

编写构建Fiber的逻辑，在getFirstTask方法中首先要获取subTask，从任务队列中获取这个任务。然后构建最外层节点对应的Fiber对象。

这里我首先要获取的是第一个小任务，并不是任务队列中的第一个任务，是任务队列中任务的第一个小任务。把任务队列中拿出来的task看成是一个大任务，要从中获取第一个子任务。也就是构建最外层Fiber节点对象。这里返回这个Fiber对象。

```js
{
    type: "节点类型",
    props: "节点属性",
    stateNode: "节点DOM对象 或 组件实例对象",
    tag: "节点标记", // host_root  host_component class_component function_component
    effects: ['存储需要更改的fiber对象'],
    effectTag: "当前Fiber要被执行的操作", // 新增 删除 修改
    parent: "当前Fiber的父级Fiber",
    child: "当前Fiber的子级Fiber",
    sibling: "当前Fiber的兄弟Fiber",
    alternate: "Fiber备份用于比对时使用"
}
```

对于最外层节点不需要有type属性，所以我们直接省略掉。因为是最外层节点，所以effectTag和parent也是不存在的，sibling和alternate暂时也不需要，child暂时先写成null。

```js
const getFirstTask = () => {
    // 从任务队列中获取任务
    const task = taskQueue.pop();
    // 返回最外层节点Fiber对象
    return {
        props: task.props,
        stateNode: task.dom,
        tag: 'host_root',
        effects: [],
        child: null,
    }
}
```

getFirstTask函数返回的值会赋值给subTask，当subTask有值并且浏览器有空余时间的时候就会调用循环，执行executeTask函数。父级节点已经构建完了，这个时候我们要开始构建子级节点了。executeTask接收的参数就是subTask。

要构建子节点首先要拿到子节点的虚拟DOM对象，可以通过subTask.props.children, subTask也就是fiber，所以可以从fiber.props.children中获取。

```js
const executeTask = fiber => {
    reconcileChildren(fiber, fiber.props.children)
}
```

这里我们调用reconcileChildren方法来做这件事，传入fiber和子节点的虚拟DOM对象。因为要建立父子之间的关系，所以两个参数都要传递。

这里的children可能是一个DOM对象也可能是一个数组，在构建子节点之前我们要处理一下。我们统一处理成数组，如果是对象就给对象包裹一层数组。

接着我们要拿到arrifiedChildren中的虚拟DOM将它转换成Fibler，这里用到了循环。这里定义了三个变量，index和number用于循环，element存储当前遍历到的DOM对象。

```js
const arrified = arg => Array.isArray(arg) ? arg : [arg]

const reconcileChildren = (fiber, children) => {
    // 将children转换成数组
    const arrifiedChildren = arrified(children);

    let index = 0;

    let numberOfElements = arrifiedChildren.length;

    let element = null;

    while (index < numberOfElements) {
        element = arrifiedChildren[index];
        index++;
    }
}
```

element就是我们需要的子节点，接下来我们就可以构建fiber对象了，我们声明一个newFiber对象。他的值默认为空，当循环的时候获取到对应的Fiber对象。

```js
const reconcileChildren = (fiber, children) => {
    // 将children转换成数组
    const arrifiedChildren = arrified(children);

    let index = 0;

    let numberOfElements = arrifiedChildren.length;

    let element = null;
    // 当前正在构建的的Fiber
    let newFiber = null;
    // 存储前一个节点，用于构建兄弟关系
    let prevFiber = null;

    while (index < numberOfElements) {
        element = arrifiedChildren[index];
        newFiber = {
            type: element.type,
            props: element.props,
            tag: 'host_component',
            effects: [],
            effectTag: 'placement', // 新增
            stateNode: null, // dom对象，暂时没有
            parent: fiber,
        }
        // 如果第一个子节点就赋值到fiber上
        if (index == 0) {
            fiber.child = newFiber;
        } else {
            // 否则放在前一个的兄弟节点上
            prevFiber.sibling = newFiber;
        }
        prevFiber = newFiber;
        index++;
    }
}
```

在fiber对象当中有一个属性叫做stateNode, 这个属性的值要取决当前节点的类型，如果当前节点是普通节点，就存储当前节点DOM对象，如果是组件的话就存储组件的实例对象。这里我们要声明一个方法来做这个判断。这个方法接收当前的fiber对象作为参数。

这里依赖createDOMElement方法。

```js
// 获取节点对象
newFiber.stateNode = createStateNode(newFiber);

const createStateNode = fiber => {
    // 普通节点
    if (fiber.tag === 'host_component') {
        return createDOMElement(fiber)
    }
}
```

在Fiber对象中还有一个tag属性，他表示的是当前节点到底是标签节点还是组件节点，这里也需要一个函数来判断。

```js

const getTag = vdom => {
    if (typeof vdom.type === 'string') {
        return 'host_component'
    }
}

newFiber = {
    type: element.type,
    props: element.props,
    tag: getTag(element),
    effects: [],
    effectTag: 'placement', // 新增
    stateNode: null, // dom对象，暂时没有
    parent: fiber,
}

```

这里需要注意的是根节点是不需要调用getTag来获取的，根节点始终都为host_root字符串。

至此外层节点对象和第一个子集节点对象我们就构建完成了，接着我们开始查找节点，继续构建节点对象。之前我们通过reconcileChildren方法构建了外层节点对象和第一个子级节点对象，当第一个子集构建完成之后，代码会重新回到这个函数。我们在这里判断fiber是否有child，如果有就返回。这样executeTask执行结束之后就会返回一个新的任务。

这样executeTask执行完会再次执行executeTask。第二次执行时传入的fiber就是fiber.child，将子级当做父级来执行。继续构建子级的子级。

```js
const executeTask = fiber => {
    // 构建子级fiber对象
    reconcileChildren(fiber, fiber.props.children)
    if (fiber.child) {
        return fiber.child
    }
}

const workLoop = deadline => {
    // 如果子任务不存在获取一个任务
    if (!subTask) {
        subTask = getFirstTask()
    }
    // 任务存在并且浏览器空余时间大于1ms执行任务
    while (subTask && deadline.timeRemaining() > 1) {
        subTask = executeTask(subTask);
    }
}
```

下面我们继续构建其他节点的Fiber对象，上面的代码我们已经构建完了一条链路的节点，这个时候定位的肯定是一条链路的最后一个节点，我们要根据这个节点查找其他节点，去构建其他节点的fiber对象。

查找原则很简单，如果当前节点有同级就去构建同级的子级，如果当前节点没有同级，就去查看父级是否有同级，这样一直查找就可以把所有的节点构建完了。当最终退回到了根节点就证明构建完了。

```js
const executeTask = fiber => {
    // 构建子级fiber对象
    reconcileChildren(fiber, fiber.props.children)
    // 有子级返回子级
    if (fiber.child) {
        return fiber.child
    }
    // 存储当前正在处理的对象
    let currentExecutelyFiber = fiber;

    while (currentExecutelyFiber.parent) {
        // 有同级返回同级
        if (currentExecutelyFiber.sibling) {
            return currentExecutelyFiber.sibling
        }
        // 没有同级将父级给到循环，循环检查父级
        currentExecutelyFiber = currentExecutelyFiber.parent
    }
}
```

这样我们就已经找到了所有节点，并且为所有节点构建Fiber对象了。

## 构建effects

在fiber算法的第二阶段，要循环遍历所有fiber构建真实DOM对象，并且将构建出来的DOM对象添加到页面中，所以我们要将所有fiber对象存储到一个数组中，方便操作。

effects数组就是存储fiber对象的，我们要将所有的fiber对象存储在最外层节点对象的effects数组中。

所有的节点都有effects对象，最外层节点负责存储所有的fiber对象，其他节点负责收集fiber对象。然后汇总到最外层。我们可以在循环代码中加入这段逻辑。

在while循环中currentExecutelyFiber是正在操作的fiber对象，可以找到父级的effects数组。我们让这个数组等于自有的值加上当前currentExecutelyFiber的effects的值加上当前的currentExecutelyFiber。

```js
while (currentExecutelyFiber.parent) {
    // 当前父级的effects存储当前的effects和当前对象。
    currentExecutelyFiber.parent.effects = currentExecutelyFiber.parent.effects.concat(currentExecutelyFiber.effects.concat(currentExecutelyFiber))
    // 有同级返回同级
    if (currentExecutelyFiber.sibling) {
        return currentExecutelyFiber.sibling
    }
    // 没有同级将父级给到循环，循环检查父级
    currentExecutelyFiber = currentExecutelyFiber.parent
}
```

这样我们就将所有的子级对象存储到父级的effects里面了。

## 实现初始渲染

当循环执行完毕的时候currentExecutelyFiber存储的就是最外层的节点对象，通过他的effects我们就可以拿到所有节点的fiber对象。接着我们就可以进入到fiber算法的第二个阶段了构建DOM节点的关系，构建完成之后添加到页面中。

这里首先我们要把currentExecutelyFiber改成全局变量，因为我们要传递下去。定义pendingCommit来接收这个变量。

```js
const executeTask = fiber => {
    ...
    pendingCommit = currentExecutelyFiber;
}
```

当fiber构建完成之后会执行到workLoop函数的循环下面，我们在这里判断pendingCommit是否存在，如果存在就调用方法传入这个参数。

```js
const workLoop = deadline => {
    // 如果子任务不存在获取一个任务
    if (!subTask) {
        subTask = getFirstTask()
    }
    // 任务存在并且浏览器空余时间大于1ms执行任务
    while (subTask && deadline.timeRemaining() > 1) {
        subTask = executeTask(subTask);
    }
    // 执行第二阶段
    if (pendingCommit) {
        commitAllWork(pendingCommit)
    }
}
```

在commitAllWork这个方法中我们循环effects数组, 如果他的effectTag类型是placement也就是新增节点的时候，我们就将它append到父级中。

```js
const commitAllWork = fiber => {
    fiber.effects.forEach(item => {
        // 追加节点
        if (item.effectTag === 'placement') {
            item.parent.stateNode.appendChild(item.stateNode);
        }
    })
}
```

## 渲染类组件

我们要把类组件当中返回的内容渲染到页面中，我们getTag方法中需要返回这种类型。

```js
const getTag = vdom => {
    // 如果是普通节点
    if (typeof vdom.type === 'string') {
        return 'host_component'
    } else if (Object.getPrototypeOf(vdom.type) === Component) {
        // 如果是类组件
        return 'class_component'
    } else {
        // 函数组件
        return 'function_component'
    }
}
```

接着我们还要去处理stateNode属性，如果他是普通节点就存储节点DOM对象，如果是组件就存储组件的实例对象，在createStateNode这个方法中处理这种逻辑。

如果是组件分为类组件和函数组件，我们也要通过一个方法来判断。createReactInstance()。

```js
const createStateNode = fiber => {
    // 普通节点
    if (fiber.tag === 'host_component') {
        return createDOMElement(fiber)
    } else {
        // 组件
        return createReactInstance()
    }
}
```

区分函数组件和类组件在fiber对象中有个tag属性，如果他的值是class_component就是类组件否则就是函数组件。如果是类组件就返回构造函数, 如果是函数组件就直接等于type就可以了。

```js
createReactInstance = fiber => {
    let instance = null;
    
    if (fiber.tag === 'class_component') {
        instance = new fiber.type(fiber.props)
    } else {
        instance = fiber.type;
    }
    return instance;
}
```

现在我们就将stateNode值和tag值处理好了，接下来我们在调用reconcileChildren方法的时候父级参数应该是fiber，子级参数应该是render方法中的返回值。所以我们在执行这个方法的时候不能直接传fiber.props.children，所以在executeTask函数调用reconcileChildren之前做一个判断，根据fiber的tag类型来处理。

```js
const executeTask = fiber => {
    // 构建子级fiber对象
    if (fiber.tag === 'class_component') {
        reconcileChildren(fiber, fiber.stateNode.render())
    } else {
        reconcileChildren(fiber, fiber.props.children)
    }
    ...
}
```

接着在commitAllWork方法中需要继续处理，类组件本身也是节点，但是类组件是不能追加DOM对象的，因为类组件本身就不是一个有效的DOM元素，我们要找到类组件的父级节点，他的父级一定要是普通的DOM元素，如果不是就继续查找直到找到，我们要往这个普通的DOM元素的父级里面去追加类组件返回的内容。

```js
const commitAllWork = fiber => {
    fiber.effects.forEach(item => {
        // 追加节点
        if (item.effectTag === 'placement') {
            let fiber = item;
            let parentFiber = item.parent
            while (parentFiber.tag === 'class_component') {
                parentFiber = parentFiber.parent;
            }
            if (fiber.tag === 'host_component') {
                parentFiber.stateNode.appendChild(fiber.stateNode);
            }
        }
    })
}
```

## 渲染函数组件

在函数组件节点的Fiber中，stateNode存储的是函数组件的函数本身，tag存储的是function_component字符串，这个我们前面已经写完了。接下来我们要找到调用的地方也就是executeTask，当我们处理函数组件的时候他的子节点需要通过调用函数组件得到，我们添加一个else，如果处理的是函数组件。

```js
 if (fiber.tag === 'class_component') {
    reconcileChildren(fiber, fiber.stateNode.render())
} else if (fiber.tag === 'function_component') {
    reconcileChildren(fiber, fiber.stateNode(fiber.props))
} else {
    reconcileChildren(fiber, fiber.props.children)
}
```

接着我们进入到commitAllWork中，我们循环中不光要判断父级是类组件，还要判断是函数组件，如果是函数组件他本身也不能追加真实的DOM元素，同样要寻找普通DOM节点的祖先级。

```js
while (parentFiber.tag === 'class_component' || parentFiber.tag === 'function_component') {
    parentFiber = parentFiber.parent;
}
```

我们也可以测试一下传递props参数，都是可以运行的。

## 实现节点更新

当DOM节点初始化完成之后我们要去备份旧的Fiber节点对象，再更新的时候我们要去查看旧的Fiber节点是否存在，如果存在就是更新操作，就去创建执行更新的Fiber节点对象。

我们首先来备份节点对象，在reconciliation的commitAllWork方法中当DOM执行操作完成之后就可以备份Fiber节点对象了。我们将它备份到真实的DOM根节点对象中就可以了。

```js
fiber.stateNode.__rootFiberContainer = fiber;
```

接下来我们去创建fiber节点对象的getFirstTask方法中，我们需要添加一个alternate属性，存储的就是备份fiber对象。这样在新的根节点对象中就有这个备份了。

```js
const getFirstTask = () => {
    // 从任务队列中获取任务
    const task = taskQueue.pop();
    // 返回最外层节点Fiber对象
    return {
        props: task.props,
        stateNode: task.dom,
        tag: 'host_root',
        effects: [],
        child: null,
        alternate: task.dom.__rootFiberContainer
    }
}
```

接着我们要在reconcileChildren方法中判断要执行什么样的操作，从而去构建不同操作对应的fiber对象。我们先把备份节点获取到。

如果有fiber.alternate对象就获取子节点，从fiber.alternate.child获取，表示子节点。这个时候fiber.alternate.child是备份子节点，while第一次循环的时候从children中获取到的element就是当前的子节点。

```js
let alternate = null;
// 如果有对象就获取子节点
if (fiber.alternate && fiber.alternate.child) {
    // 如果找得到这个子节点就是children数组中的第一个节点的备份节点
    alternate = fiber.alternate.child;
}
while (index < numberOfElements) {
    // 子级虚拟DOM对象
    element = arrifiedChildren[index];
}
```

同时我们需要知道，只有第一个子节点是child，其余的都是sibling，所以我们更新alternate的时候要取sibling。

```js
// 更新alternate
if (alternate && alternate.sibling) {
    alternate = alternate.sibling;
} else {
    alternate = null;
}
```

如果element存在，alternate不存在就是初始渲染, 如果都存在就是更新操作。更新操作要创建执行更新的Fiber对象，effectTag变为update, 还要把alternate存储起来。

```js
if (element && alternate) {
    // 更新操作
    newFiber = {
        type: element.type,
        props: element.props,
        tag: getTag(element),
        effects: [],
        effectTag: 'update', // 新增
        stateNode: null, // dom对象，暂时没有
        parent: fiber,
        alternate,
    }
    if (element.type === alternate.type)
    newFiber.stateNode = createStateNode(newFiber);
}
```

还要判断要更新的和备份的类型是否相同。如果类型不同就不做比对了，直接创新新的替换老的就可以的，如果类型相同就要进行比对操作。

```js
if (element.type === alternate.type) {
    // 类型相同
    newFiber.stateNode = alternate.stateNode;
} else {
    // 类型不同
    newFiber.stateNode = createStateNode(newFiber);
}
```

接着我们要在commitAllWork这个方法中增加更新的逻辑，如果item.effectTag是update就是更新。更新操作要判断节点相同和节点不同，所以我们也分为两种情况。

```js
if (item.effectTag === 'update') {
    // 更新操作
    if (item.type === item.alternate.type) {
        // 节点类型相同
    } else {
        // 节点类型不同
    }
} else if (item.effectTag === 'placement') {
```

如果节点类型不同使用新节点替换旧节点就可以了，我们需要找到父级节点来操作。调用replaceChild, 如果节点类型相同直接更新操作就可以了。updateNodeElement第一个参数是要更新的DOM节点，第二个参数是新的虚拟DOM，第三个参数是旧的虚拟DOM。

```js
// 更新操作
if (item.type === item.alternate.type) {
    // 节点类型相同
    updateNodeElement(item.stateNode, item, item.alternate);
} else {
    // 节点类型不同
    item.parent.stateNode.replaceChild(item.stateNode, item.alternate.stateNode);
}
```

我们这个代码还有一点问题，这里只能更新元素节点，不能更新文本节点我们还要完善updateNodeElement方法实现更新文本节点。

我们在上部添加一段处理文本节点的逻辑，如果是文本节点就不让他向下执行了，因为下面的代码是执行元素节点的。

我们要在这里面查看新的文本节点和旧的文本节点是否相同，如果不同就去更新。我们还需要判断一下当前的节点父级是否相同，如果相同再进行这种操作，如果不同需要额外处理。

```js
if (virtualDOM.type === 'text') {
    if (newProps.textContent !== oldProps.textContent) {
        // 判断父级节点是否相同，如果不同就将当前文本追加到父级
        if (virtualDOM.parent.type !== oldVirtualDOM.parent.type) {
            virtualDOM.parent.stateNode.appencChild(document.createTextNode(newProps.textContent))
        } else {
            // 新的文本节点替换旧的文本节点
            virtualDOM.parent.stateNode.replaceChild(document.createTextNode(newProps.textContent), oldVirtualDOM.stateNode);
        }
    }
    return;
}
```

## 实现节点删除

我们要在reconcileChildren方法中判断当前执行的是否是删除操作。如果是删除就去构建执行删除操作的Fiber节点对象，我们只需要判断element是否存在，如果element不存在，备份节点确存在那就是删除操作。

```js
// 如果element不存在，alternate存在，就是删除
if (!element && alternate) {
    // 删除节点
    alternate.effectTag = 'delete'
    // 添加到父级的effects数组中
    fiber.effects.push(alternate);
}
```

我们要更新循环条件，如果是空数组这个循环就进不来了，但是alternate是有可能存在的，所以要改成。

```js
while (index < numberOfElements || alternate) {
```

如果element不存在的话newFiber也是不存在的，所以我们在下面也要判断一下。

```js
// 如果第一个子节点就赋值到fiber上
if (index == 0) {
    fiber.child = newFiber;
} else if (element) {
    // 否则放在前一个的兄弟节点上
    prevFiber.sibling = newFiber;
}
```

最后我们要在commitAllWork中执行真正的DOM操作来删除节点，如果item.effectTag === 'delete'当前执行的就是删除操作。

```js
if (item.effectTag === 'delete') {
    item.parent.stateNode.removeChild(item.stateNode);
} else if (item.effectTag === 'update') {
```

## 类组件状态更新

当组件状态发生改变的时候我们要将这个任务放在任务队列中，还要指定当浏览器空闲的时候执行，我们在Component类中添加setState方法，这个方法调用scheduleUpdate方法来进行更新。

```js
class Component {
    constructor(props) {
        this.props = props;
    }

    setState(partialState) {
        scheduleUpdate(this, partialState)
    }
}
```

scheduleUpdate定义在reconciliation这个js中，目的是为了和之前的方法进行互相调用。这个方法接收组件实例对象和新的state两个参数。

通过实例对象获取state，使用新的状态覆盖他就可以了。别忘了将这个更新的任务添加到任务队列中。from定义一种新的任务类型便于区分。

最后别忘了添加requestIdleCallback(performTask)在浏览器空闲的时候执行任务队列。

```js
const scheduleUpdate = (instance, partialState) => {
    taskQueue.push({
        from: 'class_component',
        instance,
        partialState
    })
    requestIdleCallback(performTask)
}
```

在getFirstTask获取任务这个方法中处理这种类型的任务。这里先返回不让他向下执行，要处理新的任务。

```js
const getFirstTask = () => {
    // 从任务队列中获取任务
    const task = taskQueue.pop();
    if (task.from === 'class_component') {
        return;
    }
    // 返回最外层节点Fiber对象
    return {
        props: task.props,
        stateNode: task.dom,
        tag: 'host_root',
        effects: [],
        child: null,
        alternate: task.dom.__rootFiberContainer
    }
}
```

在commitAllWork的循环中要判断一下这里处理的是否是类组件，根据tag为class_component来判断。如果是类组件就在这个组件的实例对象身上添加备份属性存储fiber。

```js
if (item.tag === 'class_component') {
    // 处理的是类组件
    item.stateNode.__fiber = item;
}
```

现在我们就在组件的实例对象上备份了组件的fiber对象。这样我们再getFirstTask的判断中就可以拿到组件的fiber对象了。task.instance.__fiber。

```js
if (task.from === 'class_component') {
    const root = getRoot(task.instance);
    return {
        props: root.props,
        stateNode: root.stateNode,
        tag: 'host_root',
        effects: [],
        child: null,
        alternate: root // root就是备份
    };
}
```

我们向上查找直到找到最外层节点对象。

```js
const getRoot = instance => {
let fiber = instance.__fiber
while (fiber.parent) {
    fiber = fiber.parent
}
return fiber;
}
```

下面我们就要去更新状态了，在更新之前我们需要把要更新的状态存储起来

```js
const root = getRoot(task.instance);
// 将要更新的状态存储起来
task.instance.__fiber.partialState = task.partialState;
```

在executeTask方法中，如果是类组件，我们就看partialState是否存在，如果存在就更新组件。

```js
// 构建子级fiber对象
if (fiber.tag === 'class_component') {
    // 更新state
    if (fiber.stateNode.__fiber && fiber.stateNode.__fiber.partialState) {
        fiber.stateNode.state = {
            ...fiber.stateNode.state,
            ...fiber.stateNode.__fiber.partialState
        }
    }
    reconcileChildren(fiber, fiber.stateNode.render())
} else if (fiber.tag === 'function_component') {
```
