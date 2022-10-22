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

在while循环中currentExecutelyFiber是正在操作的fiber对象，可以找到父级的effects数组。我们让这个数组等于自有的值加上当前currentExecutelyFiber的effe