JSX只是看起来像HTML实际上它是JavaScript，在React代码执行之前Babel会将JSX编译为React API。

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

React.createElement用来创建虚拟DOM，返回的是虚拟DOM对象。React将虚拟DOM转换为真实DOM最终显示到页面中。

jsx在运行时会被Babel转换为React.createElement对象，React.createElement会被React转换成虚拟DOM对象，虚拟DOM对象又会被React转换成真实DOM对象。JSX语法的出现就是为了让React开发人员编写用户界面代码更加轻松。

## 1. 虚拟DOM

在React中，每个DOM对象都有一个对应的虚拟DOM对象，其实就是使用JavaScript对象来描述DOM对象信息，比如DOM对象的类型，属性，子元素。

```jsx
// 编译前
<div className="content">
    <h3>Hello React</h3>
    <p>React is great</p>
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
        },
        {
            type: "p",
            props: null,
            children: [
                {
                    type: "text",
                    props: {
                        textContent: "React is greate"
                    }
                }
            ]
        }
    ]
}
```

React采用最小化的DOM操作来提升DOM操作的优势，只更新需要更新的，在React第一次创建DOM对象的时候会为每一个DOM对象创建虚拟的DOM对象，在DOM对象发生更新之前React会更新所有的虚拟DOM对象, 然后将更新前的虚拟DOM和更新后的虚拟DOM进行对比，找到变更的DOM对象，只将发生变化的DOM更新到页面中从而提升了js操作DOM的性能。

虽然在操作真实DOM之前进行的虚拟DOM更新和对比的操作，但是由于JS操作自有对象效率是很高的，成本几乎可以忽略不计的。

在React代码执行前，JSX会被Babel转换为React.createElement方法的调用，在调用createElement方法时会传入元素的类型，属性，以及子元素，返回值为构建好的虚拟DOM对象。

```js
function createElement(type, props, ...children) {
    return {
        type,
        props,
        children
    }
}
```

使用TinyReact分析React代码。首先配置babel将jsx编译为Tiny的createElement方法。.babelrc

```JSON
{
    "presets": [
        "@babel/preset-env",
        [
            "@babel/preset-react",
            {
                "pragma": "TinyReact.createElement"
            }
        ]
    ]
}
```

src/index.js

```jsx
import TinyReact from "./TinyReact"

const virtualDOM = (
  <div className="container">
    <h1>你好 我是虚拟DOM</h1>
  </div>
)

console.log(virtualDOM);
```

控制台打印结果。

```JSON
{
    "type": "div",
    "props": {
        "className": "container"
    },
    "children": [
        {
            "type":"h1",
             "props":null,
            "children": [
                "你好 我是虚拟DOM"
            ]
        }
    ]
}
```

这里就打印出来一个简单的虚拟DOM，这里的文本节点"你好 我是虚拟DOM"正确的做法应该是文本节点也应该是一个虚拟DOM对象。

```js
function createElement(type, props, ...children) {
    // 遍历children对象
    const childElements = [].concat(...children).map(child => {
        if(child instanceof Object) {
        return child; // 是对象直接返回
        } else {
        // 不是对象 调用createElement方法生成一个对象
        return createElement('text', { textContent: child });
        }
    })
    return {
    type,
    props,
    children: childElements
    }
}

{
    "type": "div",
    "props": {
        "className": "container"
    },
    "children": [
        {
            "type":"h1",
             "props":null,
            "children": [
                {
                    "type":"text",
                    "props": {
                        "textContent": "你好 我是虚拟DOM"
                    },
                    "children": []
                }
            ]
        }
    ]
}
```

在组件模板中如果是布尔值或者null值，节点是不显示的。

```jsx
<div className="container">
    <h1>你好 我是虚拟DOM</h1>
    {
        1 === 2 && <h1>布尔值节点</h1>
    }
</div>
```

```js
function createElement(type, props, ...children) {
  // 遍历children对象
  const childElements = [].concat(...children).reduce((result, child) => {
    // 判断child不能是布尔也不能是null
    // 因为使用reduce，所以result是前一次循环的返回值，最终返回result就可以
    if (child !== false && child !== true && child !== null) {
      if (child instanceof Object) {
        result.push(child); // 是对象直接返回
      } else {
        // 不是对象 调用createElement方法生成一个对象
        result.push(createElement('text', {
          textContent: child
        }));
      }
    }
    return result;
  }, [])
  return {
    type,
    props,
    children: childElements
  }
}
```

将children放入到props中。

```js
return {
    type,
    props: Object.assign({ children: childElements}, props),
    children: childElements
}
```

## 2. 虚拟DOM转换为真实DOM

定义render方法，src/tinyReact/render.js接收三个参数，第一个参数是虚拟DOM，第二个参数是要渲染到的页面元素，第三个参数是旧的虚拟DOM用于进行对比。主要作用就是将虚拟DOM转换为真实DOM并且渲染到页面中。

```js
import diff from './diff'

function render(virtualDOM, container, oldDOM) {
    diff(virtualDOM, container, oldDOM);
}
```

需要在diff方法中进行一次处理，如果旧的虚拟DOM存在就进行对比，如果不存在就直接将当前的虚拟DOM放置在container中。src/tinyReact/diff.js

```js
import mountElement from './mountElement';

function diff (virtualDOM, container, oldDOM) {
    // 判断oldDOM是否在巡
    if (!oldDOM) {
        return mountElement(virtualDOM, container);
    }
}
```

判断需要转换的虚拟DOM是组件还是普通的标签。需要分别进行处理, 这里先默认只有原生jsx标签，写死调用mountNativeElement方法。src/tinyReact/mountElement.js

```js
import mountNativeElement from './mountNativeElement';

function mountElement(virtualDOM, container) {
    // 处理原生的jsx和组件的jsx
    mountNativeElement(virtualDOM, container);
}
```


mountNativeElement文件用于将原生的虚拟DOM转换成真实的DOM，调用createDOMElement方法来实现。src/tinyReact/mountNativeElement.js

```js
import createDOMElement from './createDOMElement';

function mountNativeElement(virtualDOM, container) {
    // 将虚拟dom转换成真实的对象
    let newElement = createDOMElement(virtualDOM);
    // 将转换之后的DOM对象放在页面中
    container.appendChild(newElement);
}
```

判断如果是元素节点就创建相应的元素，如果是文本节点就创建对应的文本。通过递归的方式创建子节点。最后将创建的节点放在指定的容器container中。src/tinyReact/createDOMElement.js

```js
import mountElement from "./mountElement";

function createDOMElement(virtualDOM) {
    let newElement = null;
    if (virtualDOM.type === 'text') {
        // 文本节点 使用createTextNode创建
        newElement = document.createTextNode(virtualDOM.props.textContent);
    } else {
        // 元素节点 使用 createElement 创建
        newElement = document.createElement(virtualDOM.type);
    }
    // 递归创建子节点
    virtualDOM.children.forEach(child => {
        mountElement(child, newElement);
    })
    return newElement;
}
```

## 3. 为DOM添加属性

属性是存储在虚拟DOM的props中的，在创建元素的时候循环这个属性，将这些属性放在真实的元素中。

添加属性的时候需要考虑不同的情况，比如说事件和静态属性都是不同的，而且添加属性的方法也是不同的，布尔属性和值属性的设置方式有所不同。还需要判断属性是不是children，因为children并不是属性，属性如果是className还需要转换成class进行添加。

src/tinyReact/createDOMElement.js

单独定一个方法来为元素添加属性，在创建元素之后调用这个方法，这里叫做updateNodeElement

```js
import mountElement from "./mountElement";
import updateNodeElement from "./updateNodeElement";

function createDOMElement(virtualDOM) {
    let newElement = null;
    if (virtualDOM.type === 'text') {
        // 文本节点 使用createTextNode创建
        newElement = document.createTextNode(virtualDOM.props.textContent);
    } else {
        // 元素节点 使用 createElement 创建
        newElement = document.createElement(virtualDOM.type);
        // 调用添加属性的方法
        updateNodeElement(newElement, virtualDOM)
    }
    // 递归创建子节点
    virtualDOM.children.forEach(child => {
        mountElement(child, newElement);
    })
    return newElement;
}
```

使用Object.keys来获得属性名使用forEach来遍历。

src/tinyReact/updateNodeElement.js

如果属性名以on开头就认为是一个事件，使用addEventListener来绑定事件。

如果属性名是value或者checked是不能使用setAttribute来设置的，直接属性名等于属性值即可。

如果是className就转换成class，如果不为children则其它属性全部可以使用setAttribute来设置。

```js
function updateNodeElement(newElement, virtualDOM) {
    // 获取节点对应的属性对象
    const newProps = virtualDOM.props;
    Object.keys(newProps).forEach(propName => {
        const newPropsValue = newProps[propName];
        // 判断是否是事件属性
        if (propName.startsWith('on')) {
            // 截取出事件名称
            const eventName = propName.toLowerCase().slice(2);
            // 为元素添加事件
            newElement.addEventListener(eventName, newPropsValue);
        } else if (propName === 'value' || propName === 'checked') {
            // 如果属性名是value或者checked不能使用setAttribute来设置，直接以属性方式设置即可
            newElement[propName] = newPropsValue;
        } else if (propName !== 'children') {
            // 排除children
            if (propName === 'className') {
                newElement.setAttribute('class', newPropsValue)
            } else {
                newElement.setAttribute(propName, newPropsValue)
            }
        }
    })
}
```

## 4. 组件渲染

组件的虚拟DOM类型值为函数，函数组件和类组件都是如此。

渲染组件时，要先将Component与Native Element区分开，如果是Native Element可以直接进行渲染，如果是组件需要特别处理。在入口文件src/index.js中渲染一个组件。

```jsx
import TinyReact from "./TinyReact"

const root = document.getElementById('root');

function Demo () {
    return <div>hello</div>
}
function Head () {
  return <div><Demo /></div>
}

TinyReact.render(<Head />, root);
```

在mountElement方法中区分原生标签和组件。

src/tinyReact/isFunction.js

```js
function isFunction(virtualDOM) {
    return virtualDOM && typeof virtualDOM.type === 'function';
}
```

如果type存在，并且对象是一个函数，并且对象上不存在render方法，那就是一个函数组件

src/tinyReact/mountComponent.js

```js
import isFunctionComponent from './isFunctionComponent';

function mountComponent(virtualDOM, container) {
    // 判断组件是类组件还是函数组件
    if (isFunctionComponent(virtualDOM)) {
        
    }
}
```

src/tinyReact/isFunctionComponent.js

```js
import isFunction from "./isFunction";

function isFunctionComponent(virtualDOM) {
    const type = virtualDOM.type;
    return type && isFunction(virtualDOM) && !(type.prototype && type.prototype.render)
}
```

函数组件其实很简单，只需要调用type函数就可以获取返回的虚拟dom。获取之后我们需要判断新获取的虚拟DOM是否是一个组件，如果是继续调用mountComponent，如果不是则为原生DOM元素直接调用mountNativeElement方法将虚拟DOM渲染到页面中。

src/tinyReact/mountComponent.js

```js
import isFunction from './isFunction';
import isFunctionComponent from './isFunctionComponent';
import mountNativeElement from './mountNativeElement';

function mountComponent(virtualDOM, container) {
    //存储得到的虚拟DOM
    let nextVirtualDOM = null;
    // 判断组件是类组件还是函数组件
    if (isFunctionComponent(virtualDOM)) {
        // 处理函数组件
        nextVirtualDOM = buildFunctionComponent(virtualDOM);
    }
    // 判断是否仍是一个函数组件
    if (isFunction(nextVirtualDOM)) {
        mountComponent(nextVirtualDOM, container);
    }
    // 渲染nextVirtualDOM
    mountNativeElement(nextVirtualDOM, container);
}

function buildFunctionComponent (virtualDOM) {
    return virtualDOM.type();
}
```

在组件的身上是有一个props参数的，在组件的内部可以在props上面拿到这个值的，当渲染函数组件的时候可以在将props传入进去。

```js
function buildFunctionComponent (virtualDOM) {
    return virtualDOM.type(virtualDOM.props || {});
}
```

渲染类组件同样是在mountComponent.js中来实现。

```js
if (isFunctionComponent(virtualDOM)) {
    // 处理函数组件
    nextVirtualDOM = buildFunctionComponent(virtualDOM);
} else {
    // 处理类组件
}
```

创建一个buildClassComponent方法来处理类组件, 这个函数接收虚拟DOM，在这个函数中需要得到组件的实例对象，因为只有得到了实例对象我们才能获得render方法，通过调用render方法才能获得组件输出的虚拟DOM对象。

```js
// 处理类组件
function buildClassComponent (virtualDOM) {
    // 获取实例对象
    const component = new virtualDOM.type();
    // 获得虚拟DOM对象
    const nextVirtualDOM = component.render();
    return nextVirtualDOM;
}
```

在类组件中可以通过this.props拿到传递的参数，类组件是集成了Component父类，可以在子类中调用父类的方法，让父类中的props等于传入的props，这样子类就可以拿到props了。

在子类中添加一个构造函数，接收props，然后调用super父类，将props传入给父类。

```jsx
class Alert extends TinyReact.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <div>{this.props.name} {this.props.age}</div>
  }
}
```

在父类的构造函数中拿到props然后赋值给props属性。这样子类继承了父类，子类也就有这个属性了。

```js
class Component {
    constructor(props) {
        this.props = props;
    }
}
```

在实例化组件的时候将props传递进来就可以了。

```js
function buildClassComponent (virtualDOM) {
    // 获取实例对象
    const component = new virtualDOM.type(virtualDOM.props || {});
    // 获得虚拟DOM对象
    const nextVirtualDOM = component.render();
    return nextVirtualDOM;
}
```

## 5. 更新DOM元素

要实现更新页面中的DOM元素，就要用到虚拟DOM对比，拿新的虚拟DOM和老的虚拟DOM进行对比，找出差异部分，将差异部分更新到页面中。

进行虚拟DOM比对时，需要用到更新后的虚拟DOM和更新前的虚拟DOM，更新后的虚拟DOM目前我们可以通过render方法进行传递，对于更新前的虚拟DOM，对应的其实就是已经在页面中显示的真实DOM，创建真实DOM对象时，就可以将虚拟DOM添加到真实DOM对象的属性中，在进行虚拟DOM对比之前，就可以通过真实DOM对象获取其对应的虚拟DOM对象，其实就是通过render方法的第三个参数获取的，container.firstChild.

首先为真实DOM添加对应的虚拟DOM对象。

```js
 // 添加虚拟DOM属性
newElement._virtualDOM = virtualDOM;
```

在render方法中传递三个参数，当前的虚拟DOM对象，要渲染到的容器对象以及老的虚拟DOM对象。

```js
function render(virtualDOM, container, oldDOM) {
    diff(virtualDOM, container, oldDOM);
}
```

第三个参数并不是render方法传递进来的，而是从页面获取到的，应该是container.firstChild对象。也就是当前容器内渲染的内容对象。

```js
function render(virtualDOM, container, oldDOM = container.firstChild) {
    diff(virtualDOM, container, oldDOM);
}
```

接着我们就可以在oldDOM中获取到老的虚拟DOM对象了，在diff算法中先获取。

```js
const oldVirtualDOM = oldDOM && oldDOM._virtualDOM;
```

如果oldVirtualDOM存在的话，首先对比两个元素的标签类型相同，如果两个元素类型相同，需要判断是文本类型节点还是元素类型节点，文本类型直接更新内容，元素类型就要更新标签的属性。

```js
import mountElement from './mountElement';
import updateTextNode from './updateTextNode';

function diff (virtualDOM, container, oldDOM) {
    // 获取老的虚拟DOM对象
    const oldVirtualDOM = oldDOM && oldDOM._virtualDOM;
    // 判断oldDOM是否在巡
    if (!oldDOM) {
        return mountElement(virtualDOM, container);
    } else if (oldVirtualDOM && virtualDOM.type === oldVirtualDOM.type) {
        // 两个元素类型相同，需要判断是文本类型节点还是元素类型节点
        // 文本类型直接更新内容
        // 元素类型就要更新标签的属性
        if (virtualDOM.type === 'text') {
            // 更新内容
            updateTextNode(virtualDOM, oldVirtualDOM, oldDOM);

        } else {
            // 更新元素属性
        }
        // 遍历子元素进行对比
        virtualDOM.children.forEach((child, i) => {
            diff(child, oldDOM, oldDOM.childNodes[i]);
        })
    }
}
```

抽出一个更新方法。在这个方法中判断内容是否相同，如果不相同就更新。

src/tinyReact/updateTextNode.js

```js
function updateTextNode (virtualDOM, oldVirtualDOM, oldDOM) {
    if (virtualDOM.props.textContent !== oldVirtualDOM.props.textContent) {
        // 更新DOM节点内容
        oldDOM.textContent = virtualDOM.props.textContent;
        // 更新老的虚拟DOM
        oldDOM._virtualDOM = virtualDOM;
    }
}
```

节点属性更新是将新旧节点属性对象进行对比，从中找到差异部分，然后将差异部分更新到节点属性上。使用updateNodeElement方法。

```js
if (virtualDOM.type === 'text') {
    // 更新内容
    updateTextNode(virtualDOM, oldVirtualDOM, oldDOM);

} else {
    // 更新元素属性
    // 要更新的哪个元素，更新的虚拟DOM，旧的虚拟DOM
    updateNodeElement(oldDOM, virtualDOM, oldVirtualDOM)
}
```

修改updateNodeElement方法，在里面添加oldVirtualDOM参数。获取到新旧节点属性对象newProps和oldProps。

循环新的属性对象的时候可以拿到属性名称，可以通过属性名称对比旧的属性值，来对比两个属性值是否相同。

```js
function updateNodeElement(newElement, virtualDOM, oldVirtualDOM) {
    // 获取节点对应的属性对象
    const newProps = virtualDOM.props || {};
    // 获取旧的属性对象
    const oldProps = oldVirtualDOM.props || {};
}
```

对比两个值是否相同，如果不相同就做更新操作。在更新操作中事件需要注意，清除原有事件。

```js
// 如果存在原有事件，需要删除掉。
if (oldPropsValue) {
    newElement.addEventListener(eventName, oldPropsValue);
}
```

如果有属性被删除了，需要删除DOM对象上的属性。可以循环oldProps，如果newProps中没有，则是被删除的。

```js
// 判断属性被删除的情况
Object.keys(oldProps).forEach(propName => {
    // 新的属性值
    const newPropsValue = newProps[propName];
    // 旧的属性值
    const oldPropsValue = oldProps[propName];
    if (!newPropsValue) {
        // 判断是否是事件属性
        if (propName.startsWith('on')) {
                // 截取出事件名称
                const eventName = propName.toLowerCase().slice(2);
                // 删除事件
                newElement.removeEventListener(eventName, oldPropsValue);
        } else if (propName !== 'children') {
            newElement.removeAttribute(propName);
        }
    }
})
```

虚拟DOM类型相同，如果是元素节点，就对比元素节点属性是否发生变化，如果是文本节点就对比文本节点内容是否发生变化。要实现对比，需要先从已存在的DOM对象中获取对应的虚拟DOM对象。

```js
const oldVirtualDOM = oldDOM && oldDOM._virtualDOM
```

判断oldVirtualDOM是否存在，如果存在则继续判断要对比的虚拟DOM类型是否相同，如果类型相同则判断节点类型是否是文本，如果是文本节点对比，就调用updateTextNode方法，如果是元素节点对比就调用updateNodeElement方法。

```js
else if (oldVirtualDOM && virtualDOM.type === oldVirtualDOM.type) {
    // 两个元素类型相同，需要判断是文本类型节点还是元素类型节点
    // 文本类型直接更新内容
    // 元素类型就要更新标签的属性
    if (virtualDOM.type === 'text') {
        // 更新内容
        updateTextNode(virtualDOM, oldVirtualDOM, oldDOM);

    } else {
        // 更新元素属性
        // 要更新的哪个元素，更新的虚拟DOM，旧的虚拟DOM
        updateNodeElement(oldDOM, virtualDOM, oldVirtualDOM)
    }
}
```

上层元素比对完成以后还需要递归比对子元素

```js
// 遍历子元素进行对比
virtualDOM.children.forEach((child, i) => {
    diff(child, oldDOM, oldDOM.childNodes[i]);
})
```

如果两个节点类型不同他们之间就没有必要进行比对了，只需要使用新的虚拟DOM生成新的DOM对象，替换旧的DOM对象就可以了。在diff.js中使用else if来处理这种情况。

```js
if (!oldDOM) {
    return mountElement(virtualDOM, container);
} else if (virtualDOM.type !== oldVirtualDOM.type && typeof virtualDOM.type !== 'function') {
    // 如果标签类型不相同，并且不是组件。
    const newElement = createDOMElement(virtualDOM);
    // 替换DOM元素
    oldDOM.parentNode.replaceChild(newElement, oldDOM);
} else if
```

## 6. 删除节点

删除节点发生在节点更新之后，并且发生在同一个父节点下的所有子节点身上，在节点更新完成以后，如果旧节点对象的数量多于新虚拟DOM节点的数量，就说明有节点需要被删除。

获取旧节点的数量，如果新旧节点数量不相同，我们就循环旧的DOM节点，然后从后向前删除，直到新旧DOM节点数量相同。
```js
// 删除节点
// 获取旧节点
const oldChildNodes = oldDOM.childNodes;
// 判断旧节点的数量
if (oldChildNodes.length > virtualDOM.children.length) {
    // 循环删除节点
    for (let i = oldChildNodes.length - 1; i > virtualDOM.children.length -1; i--) {
        oldDOM.removeChild(oldChildNodes[i]);
    }
}
```

## 7. 类组件的状态更新

要实现类组件的更新，需要实现setState方法，调用的setState应该是父类Component中的setState。当子类调用setState的时候首先要明确setState里面的this指向的是子类的实例对象。

```js
setState (state) {
    this.state = Object.assign({}, this.state, state);
}
```

当state发生改变的时候需要重新触发render方法，当state发生改变之后需要更新页面中的state，可以通过render获取到最新的虚拟DOM，然后和旧的虚拟DOM进行对比更新。

render方法可以获取到当前的虚拟DOM的，但是无法获取到页面展示的DOM，定义一个setDOM方法将页面展示的DOM存储起来。在类组件被实例化的时候将它传给setDOM。然后调用diff方法进行对比更新就可以了。

```js
class Component {
    constructor(props) {
        this.props = props;
    }
    setState (state) {
        this.state = Object.assign({}, this.state, state);
        // 获取最新的DOM对象
        const virtualDOM = this.render();
        // 获取旧的virtualDOM对象进行比对
        const oldDOM = this.getDOM();
        // 实现对比
        diff(virtualDOM, container, oldDOM);
    }
    setDOM (dom) { // 存储页面中展示的DOM对象
        this._dom = dom;
    }
    getDOM () { // 获取页面展示的DOM
        return this._dom;
    }
}
```

在组件更新时可能渲染的是同一个组件也可能渲染的是不同的组件，要在diff中判断要更新的虚拟DOM是否是组件，如果是组件在判断要更新的组件和未更新前的组件是否是同一个组件，如果不是同一个组件就不需要做组件更新操作，直接调用mountElement方法将组件返回的虚拟DOM添加到页面中。

如果是同一个组件，就执行更新组件操作，其实就是将最新的props传递到组件中，再调用组件的render方法获取组件返回的最新的虚拟DOM对象，再将虚拟DOM对象传递给diff方法，让diff方法找出差异，从而将差异更新到真实DOM对象中。在更新组件的过程中还要在不同阶段调用器不同的生命周期函数。

组件又分为多种情况，新增diffComponent方法接收四个参数，第一个参数是组件本身的虚拟DOM对象，通过它可以获取到组件最新的props，第二个参数是要更新的组件的实例对象，通过它可以调用组件的生命周期函数，可以更新组件的props属性，可以获取到组件返回的最新的虚拟DOM对象，第三个参数是要更新的DOM对象，在更新组件时，需要在已有DOM对象的身上进行修改，实现DOM最小化操作，获取旧的虚拟DOM对象，第四个参数是要更新到的容器元素。

```js
else if (typeof virtualDOM.type === 'function') {
    // 渲染是一个组件
    diffComponent(virtualDOM, oldComponent, oldDOM, container);
} else if

```

判断virtualDOM和oldComponent是否是同一个组件，只要判断他们的构造函数是否是同一个即可。

```js
function diffComponent(virtualDOM, oldComponent, oldDOM, container) {
    if (isSameComponent(virtualDOM, oldComponent)) {
        // 是同一个组件
    } else {
        // 不是同一个组件
        // 替换页面原有的对象，也就是删除原有DOM，增加新的DOM
        mountElement(virtualDOM, container, oldDOM);
    }
}

function isSameComponent(virtualDOM, oldComponent) {
    // 判断是否是同一个组件，只要判断他们的构造函数是否是同一个即可
    return oldComponent && virtualDOM.type === oldComponent.constructor;
}
```

如果不是同一个组件就替换原有的组件。需要在mountNativeElement接收oldDOM，然后删除这个DOM。

```js
function mountNativeElement(virtualDOM, container, oldDOM) {
    // 将虚拟dom转换成真实的对象
    // 判断旧的DOM对象是否存在，如果存在则删除
    if (oldDOM) {
        unmountNode(oldDOM);
    }
    let newElement = createDOMElement(virtualDOM);
    // 将转换之后的DOM对象放在页面中
    container.appendChild(newElement);
    // 获取实例对象
    const component = virtualDOM.component;
    if (component) {
        component.setDOM(newElement);
    }
}
```

如果需要更新的组件和旧组件是同一个组件，使用updateComponent方法实现。传入virtualDOM, container, oldDOM, container四个参数。

```js
function diffComponent(virtualDOM, oldComponent, oldDOM, container) {
    if (isSameComponent(virtualDOM, oldComponent)) {
        // 是同一个组件
        updateComponent(virtualDOM, container, oldDOM, container);
    } else {
        // 不是同一个组件
        // 替换页面原有的对象，也就是删除原有DOM，增加新的DOM
        mountElement(virtualDOM, container, oldDOM);
    }
}
```

需要在Comonent.js类中定义更新props的方法，updateProps。

```js
updateProps(props) {
    this.props = props;
}
```

通过oldComponent调用updateProps方法更新props。

```js
oldComponent.updateProps(virtualDOM.props);
```

更新之后获取到最新的虚拟DOM。通过diff算法进行比较更新。

```js
function updateComponent(virtualDOM, oldComponent, oldDOM, container) {
    // 组件更新
    oldComponent.updateProps(virtualDOM.props);
    // 获取最新的虚拟DOM，
    let nextVirtualDOM = oldComponent.render();
    // 更新实例
    nextVirtualDOM.component = oldComponent;
    // diff分别和更新。
    diff(nextVirtualDOM, container, oldDOM)
}
```

## 8. 组件生命周期

先在Component类中将生命周期默认添加进去。

```js
componentWillMount() {}
componentDidMount() {}
componentWillReceviceProps(nextProps) {}
shouldComponentUpdate(nextProps, nextState) {
    return nextProps !== this.props || nextState !== this.state;
}
componentWillUpdate(nextProps, nextState) {}
componentDidUpdate(prevProps, preState) {}
componentWillUnmount() {}
```

在updateComponent这个函数中先调用componentWillReceviceProps生命周期，在调用这个生命周期的时候要传入最新的props。

```js
oldComponent.componentWillReceviceProps(virtualDOM.props);
```

调用shouldComponentUpdate生命周期判断组件是否需要更新。

```js
if (oldComponent.shouldComponentUpdate(virtualDOM.props)) {
    // 组件更新
    oldComponent.updateProps(virtualDOM.props);
    // 获取最新的虚拟DOM，
    let nextVirtualDOM = oldComponent.render();
    // 更新实例
    nextVirtualDOM.component = oldComponent;
    // diff分别和更新。
    diff(nextVirtualDOM, container, oldDOM)
}
```

调用componentWillUpdate生命周期。

```js
// 生命周期
oldComponent.componentWillUpdate(virtualDOM.props);
```

在组件更新结束之后需要执行componentDidUpdate生命周期, 传入更新前的props。

```js
function updateComponent(virtualDOM, oldComponent, oldDOM, container) {
    // 生命周期
    oldComponent.componentWillReceviceProps(virtualDOM.props);
    // 判断是否更新生命周期
    if (oldComponent.shouldComponentUpdate(virtualDOM.props)) {
        // 存储更新前的props
        let prevProps = oldComponent.props;
        // 生命周期
        oldComponent.componentWillUpdate(virtualDOM.props);
        // 组件更新
        oldComponent.updateProps(virtualDOM.props);
        // 获取最新的虚拟DOM，
        let nextVirtualDOM = oldComponent.render();
        // 更新实例
        nextVirtualDOM.component = oldComponent;
        // diff分别和更新。
        diff(nextVirtualDOM, container, oldDOM)
        // 生命周期
        oldComponent.componentDidUpdate(prevProps);
    }
}
```

## 9. 实现ref功能

在创建节点时判断其虚拟DOM对象中是否存在ref属性，如果有就调用ref属性中所存储的方法并且将创建出来的DOM对象作为参数传递给ref方法，这样在渲染组件节点的时候就可以拿到元素对象并将元素对象存储为组件属性。在createDOMElement方法中添加。

```js
if (virtualDOM.props && virtualDOM.props.ref) {
    virtualDOM.props.ref(newElement);
}
```

在类组件身上也可以添加ref属性，目的是获取组件的实例对象。可以在mountComponent方法中，判断了如果当前处理的是类组件，就通过类组件返回的虚拟DOM对象中获取到实例对象，在实例对象中的props属性中寻找ref，如果存在就调用ref并且参数传入实例对象即可。

将componentDidMount生命周期函数添加上。

```js
// 用于存储实例对象
let component = null;
// 判断组件是类组件还是函数组件
if (isFunctionComponent(virtualDOM)) {
    // 处理函数组件
    nextVirtualDOM = buildFunctionComponent(virtualDOM);
} else {
    // 处理类组件
    nextVirtualDOM = buildClassComponent(virtualDOM);
    component = nextVirtualDOM.component;
}
if (isFunction(nextVirtualDOM)) {
    mountComponent(nextVirtualDOM, container);
}
if (component) {
    component.componentDidMount();
}
// 执行ref
if (component && component.props && component.props.ref) {
    omponent.props.ref(component);
}
```

## 10. key属性实现

key属性不需要全局唯一，但是在同一个父节点下的兄弟节点之间必须是唯一的。

之前删除节点的讲解中的实现方式是，从后向前依次删除，让前面的节点保持相同，删除多余的节点。这是很低效的，正确的做法是应该找到不需要的节点直接删除。使用key属性就可以达到这个效果。

在两个元素进行比对时，如果类型相同，就循环旧的DOM对象的子元素，查看其身上是否有key属性，如果有就将这个子元素的DOM对象存储在一个JavaScirpt对象中，接着循环要渲染的虚拟DOM对象的子元素，在循环过程中获取到这个子元素的key属性，然后使用这个key属性到JavaScript对象中查到DOM对象，如果能够找到就说明这个元素是已经存在的，是不需要重新渲染的，如果通过key属性找不到这个元素，就说明这个元素是新增的。

在diff算法中开始添加此功能。

```js
// 将拥有key属性的子元素放置在一个单独的对象中
const keyedElements = {};
for (let i = 0, len = oldDOM.childNodes.length; i < len; i++) {
    let domElement = oldDOM.childNodes[i];
    // 判断节点类型，元素节点才获取
    if (domElement.nodeType === 1) {
        const key = domElement.getAttribute('key')
        if (key) {
            keyedElements[key] = domElement;
        }
    }
}
```

如果存在就查看当前位置的元素是否是我们期望的元素。如果不是就插入到这个位置。

```js
// 循环要渲染的虚拟DOM的子元素，获取子元素的key属性
virtualDOM.children.forEach((child, i) => {
    const key = child.props.key;
    if (key) {
        const domElement = keyedElements[key];
        if (domElement) {
            // 查看当前位置的元素是否是我们期望的元素，如果不是就插入到这个位置
            if (oldDOM.childNodes[i] && oldDOM.childNodes[i] !== domElement ) {
                oldDOM.insertBefore(domElement, oldDOM.childNodes[i]);
            }
        }
    }
})
```

如果没有元素就是没有key，通过索引对比，如果有key就通过key做对比。

```js
let hasNoKey = Object.keys(keyedElements).length === 0;

if (hasNoKey) {
    // 遍历子元素进行对比
    virtualDOM.children.forEach((child, i) => {
        diff(child, oldDOM, oldDOM.childNodes[i]);
    })
} else {
    // 循环要渲染的虚拟DOM的子元素，获取子元素的key属性
    virtualDOM.children.forEach((child, i) => {
        const key = child.props.key;
        if (key) {
            const domElement = keyedElements[key];
            if (domElement) {
                // 查看当前位置的元素是否是我们期望的元素，如果不是就插入到这个位置
                if (oldDOM.childNodes[i] && oldDOM.childNodes[i] !== domElement) {
                    oldDOM.insertBefore(domElement, oldDOM.childNodes[i]);
                }
            }
        }
    })
}
```

如果找不到就说明这是新增的，通过mountElement方法直接渲染到页面中。

```js
if (key) {
    const domElement = keyedElements[key];
    if (domElement) {
        // 查看当前位置的元素是否是我们期望的元素，如果不是就插入到这个位置
        if (oldDOM.childNodes[i] && oldDOM.childNodes[i] !== domElement) {
            oldDOM.insertBefore(domElement, oldDOM.childNodes[i]);
        }
    }
} else {
    // 新增元素
    mountElement(child, oldDOM, oldDOM.childNodes[i])
}
```

在mountNativeElement这个方法中判断，如果oldDOM存在，使用container.insertBefore方法插入到oldDOM前面，不存在appendChild到最后。

```js
let newElement = createDOMElement(virtualDOM);
if (oldDOM) {
    container.insertBefore(newElement, oldDOM);
} else {
    container.appendChild(newElement);
}
```

## 11. 卸载节点

在节点比对的过程中，如果旧节点的数量多于要渲染的新节点的数量，就说明有节点被删除了，继续判断keyedElements对象中是否有元素，如果没有就使用索引方式删除，如果有就要使用key属性比对的方式进行删除。

实现思路是循环旧节点，在循环旧节点的过程中获取旧节点对应的key属性，然后根据key属性在新节点中查找这个旧节点，如果找到就说明这个节点没有被删除，如果没有找到就说明节点被删除了，调用卸载节点的方法删除节点即可。

在diff删除节点的时候判断hasNoKey是否有key。

```js
// 删除节点
// 获取旧节点
const oldChildNodes = oldDOM.childNodes;
// 判断旧节点的数量
if (oldChildNodes.length > virtualDOM.children.length) {
    if (hasNoKey) {
        // 循环删除节点
        for (let i = oldChildNodes.length - 1; i > virtualDOM.children.length - 1; i--) {
            unmountNode(oldChildNodes[i]);
        }
    } else {
        // 通过key属性删除节点
        // 拿旧的key去新的里面寻找，找不到就删除
        for (let i = 0; i < oldChildNodes.length; i++) {
            const oldChild = oldChildNodes[i];
            const oldChildKey = oldChild._virtualDOM.props.key;
            let found = false;
            for (let n = 0; n < virtualDOM.children.length; n++) {
                if (oldChildKey === virtualDOM.children[n].props.key) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                unmountNode(oldChild);
            }
        }
    }
}
```

如果过是组件生成的，需要调用组件的卸载生命周期函数，如果节点中包含了其他组件生成的节点，需要调用其他组件的卸载生命周期，如果节点身上有ref属性需要删除通过ref属性传递给组件的DOM节点对象，如果有事件也需要删除事件。我们可以在unmountNode函数中处理这些情况。

文本节点就直接删除。

```js
function unmountNode(node) {
    // 获取虚拟DOM对象
    const virtualDOM = node._virturalDOM;
    // 文本节点直接删除
    if (virtualDOM.type === 'text') {
        node.remove();
        return;
    }
}
```

组件生成的需要调用组件的卸载生命周期。

```js
// 判断节点是否是组件生成的。
const component = virtualDOM.component;
if (component) {
    component.componentWillUnmount();
}
```

ref属性如果有的话需要清理

```js
// 判断节点身上是否有ref属性，如果有的话需要清理
if (virtualDOM.props && virtualDOM.props.ref) {
    virtualDOM.props.ref(null)
}
```

卸载事件

```js
// 判断事件是否存在
Object.keys(virtualDOM.props).forEach(propsName => {
    if (propsName.startsWith('on')) {
        const eventName = propsName.toLocaleLowerCase().slice(0, 2);
        const eventHandler = virtualDOM.props[propsName];
        node.removeEventListener(eventName, eventHandler);
    }
})
```

递归删除为子节点。

```js
// 递归删除子节点
if (node.childNodes.length > 0) {
    for (let i = 0; i < node.childNodes.length; i++) {
        unmountNode(node.childNodes[i]);
    }
}
```

最后执行node.remove删除掉当前的节点。

```js
// 删除节点
node.remove();
```

```js
function unmountNode(node) {
    // 获取虚拟DOM对象
    const virtualDOM = node._virtualDOM;
    // 文本节点直接删除
    if (virtualDOM.type === 'text') {
        node.remove();
        return;
    }
    // 判断节点是否是组件生成的。
    const component = virtualDOM.component;
    if (component) {
        component.componentWillUnmount();
    }
    // 判断节点身上是否有ref属性，如果有的话需要清理
    if (virtualDOM.props && virtualDOM.props.ref) {
        virtualDOM.props.ref(null)
    }
    // 判断事件是否存在
    Object.keys(virtualDOM.props).forEach(propsName => {
        if (propsName.startsWith('on')) {
            const eventName = propsName.toLocaleLowerCase().slice(0, 2);
            const eventHandler = virtualDOM.props[propsName];
            node.removeEventListener(eventName, eventHandler);
        }
    })

    // 递归删除子节点
    if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
            unmountNode(node.childNodes[i]);
        }
    }
    // 删除节点
    node.remove();
}
```
