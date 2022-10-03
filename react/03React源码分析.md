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
    newElement.addEventL