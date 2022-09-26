## 1. 演示环境搭建

```s
npm install react express webpack webpack-cli webpack-node-externals babel-loader babel-core babel-preset-react babel-preset-stage-0 babel-preset-env --save
```

配置webpack.server.js文件，这个webpack是运行在服务端也就是node端的，需要添加一个target为node的键值对。

服务器端如果使用path路径是不需要打包到js中的，在浏览器端使用了path是需要打包到js中的，所以服务器端和浏览器端需要编译出来的js是完全不同的。在打包的时候要告诉webpack打包的是服务器端的代码还是浏览器端的代码，使用webpack-node-externals模块来标识。

entry入口文件是node的启动文件，这里写成./src/server/index.js，输出的output文件名称为bundle，目录在跟目录的build文件夹中。

```js
const Path = require('path');
const NodeExternals = require('webpack-node-externals');

module.exports = {
    target: 'node',
    mode: 'development',
    entry: './src/server/index.js',
    output: {
        filename: 'bundle.js',
        path: Path.resolve(__dirname, 'build')
    },
    externals: [NodeExternals()],
    module: {
        rules: [
            {
                test: /.js?$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                options: {
                    presets: ['react', 'stage-0', ['env', {
                        targets: {
                            browsers: ['last 2 versions']
                        }
                    }]]
                }
            }
        ]
    }
}
```

基于express模块来编写一个简单的服务。./src/server/index.js，引入React组件Home，让他参与编译，但并没有参与渲染。

```js
var express = require('express');
var app = express();
const Home = require('../Components/Home');
app.get('*', function(req, res) {
    res.send(`<h1>hello</h1>`);
})
var server = app.listen(3000);
```

编写.src/components/Home/index.js组件。

```js
import React from 'react';

const Home = () => {
    return <div>home</div>
}

export default Home;
```

通过入口文件可知js包含了一个React组件，一个端口为3000的express服务器，运行webpack打包的时候，编译出来的js文件就具备这样的功能。

```s
webpack --config webpack.server.js
```

打包之后在目录下会出现一个bundle.js，是可以运行的代码。node运行这个文件就启动了一个3000端口的服务器。

```s
node ./build/bundle.js
```

## 2. 渲染React组件

在/src/server/index.js中借助renderToString方法将Home组件转换为标签字符串。

```js
import express from 'express';
import Home from '../Components/Home';
import React from 'react';
import { renderToString } from 'react-dom/server';

const app = express();

const content = renderToString(<Home />);

app.get('*', function(req, res) {
    res.send(`
        <html>
            <body>${content}</body>
        </html>
    `);
})

var server = app.listen(3000);
```

```s
# 重新打包
webpack --config webpack.server.js
# 运行服务
node ./build/bundle.js
```

React的服务端渲染是建立在虚拟DOM上的服务器端渲染，而且服务端渲染会让页面的首屏渲染速度大大加快。不过服务端渲染也有弊端，客户端渲染React代码在浏览器端执行，消耗的是用户浏览器端的性能，服务器端渲染消耗的是服务器端的性能，毕竟React代码是很消耗计算性能的。

如果项目完全没有必要使用SEO优化并且访问速度已经很快的情况下，还是不要使用SSR的技术了，因为成本开销还是比较大的。

## 3. SSR同构

同构指的是代码复用. 即实现客户端和服务器端最大程度的代码复用，可以简单的理解同构就是让服务端和客户端执行一套代码，而不是分别针对两端写两套代码。比如给div绑定一个click事件，希望点击的时候可以弹出click提示。但是运行之后会发现这个事件并没有被绑定上，因为html是在服务器端渲染的，而服务端没办法绑定事件。

src/components/Home/index.js

```js
import React from 'react';

const Home = () => {
    return <div onClick={() => { alert('click'); }}>home</div>
}

export default Home;
```

一般的做法是先将页面渲染出来，然后将相同的代码在浏览器端像传统的React项目一样再去运行一遍，这样的话这个点击事件就有了。这就衍生出同构的概念，也就是一套React代码在服务器端执行一次，在客户端再执行一次。

可以在页面渲染的时候加载一个index.js, 使用app.use创建静态文件的访问路径, 这样访问的index.js就会请求到/public/index.js文件中。

```js

app.use(express.static('public'));

app.get('/', function(req, res) {
    res.send(`
        <html>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
    `);
})
```

public/index.js编写。

```js
console.log('public');
```

基于这种情况可以将React代码在浏览器中执行一次，新建/src/client/index.js。将客户端执行的代码帖进去。同构代码使用hydrate代替render。

```js
import React from 'react';
import ReactDOM from 'react-dom';

import Home from '../Components/Home';

ReactDOM.hydrate(<Home />, document.getElementById('root'));
```

还需要在根目录创建一个webpack.client.js文件。入口文件为./src/client/index.js，出口到public/index.js。使用webpack-merge插件对内容进行合并。

webpack.base.js

```js
module.exports = {
    module: {
        rules: [
            {
                test: /.js?$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                options: {
                    presets: ['react', 'stage-0', ['env', {
                        targets: {
                            browsers: ['last 2 versions']
                        }
                    }]]
                }
            }
        ]
    }
}
```

webpack.server.js

```js
const Path = require('path');
const NodeExternals = require('webpack-node-externals'); // 服务端运行webpack需要运行NodeExternals, 他的作用是将express这类node模块不被打包到js里。

const merge = require('webpack-merge');
const config = require('./webpack.base.js');

const serverConfig = {
    target: 'node',
    mode: 'development',
    entry: './src/server/index.js',
    output: {
        filename: 'bundle.js',
        path: Path.resolve(__dirname, 'build')
    },
    externals: [NodeExternals()],
}

module.exports = merge(config, serverConfig);
```

webpack.client.js

```js
const Path = require('path');
const merge = require('webpack-merge');
const config = require('./webpack.base.js');

const clientConfig = {
    mode: 'development',
    entry: './src/client/index.js',
    output: {
        filename: 'index.js',
        path: Path.resolve(__dirname, 'public')
    }
};

module.exports = merge(config, clientConfig);
```

src/server中放置的是服务端运行的代码，src/client放置的是浏览器端运行的js。

package.json文件中添加一条打包client目录的命令

```json
{
    ...
    "scripts": {
        "dev": "npm-run-all --parallel dev:**",
        "dev:start": "nodemon --watch build --exec node \"./build/bundle.js\"",
        "dev:build": "webpack --config webpack.server.js --watch",
        "dev:client": "webpack --config webpack.client.js --watch",
    }
    ...
}
```

这样启动的时候会编译client运行的文件。再去访问页面的时候就可以绑定好事件了。

```s
package.json # 项目管理文件
webpack.base.js # webpack 公共部分
webpack.server.js # 服务端webpack配置
webpack.client.js # 客户端webpack配置
src/  # 开发文件夹
    components/ # React 组件文件夹
    server/index.js # 服务端运行代码
    client/index.js # 客户端运行代码
    Routes/index.js # 路由配置
build/ # 编译后文件夹
public/ # 前端静态文件夹，用于存放浏览器执行的js代码
```

## 4. 渲染路由

过去的方法是浏览器向服务器发送请求，服务器返回一个空的html，浏览器再请求js，加载到js后会执行react代码，react代码接管页面执行流程，这个时候可以根据浏览器的地址展示页面内容。做重构的时候需要让路由代码在浏览器和服务端分别执行一次，浏览器执行的流程和原本一模一样没有任何区别但是服务器端有一些区别，要使用StaticRouter组件替代浏览器的browserRouter。

```s
npm install react-router-dom --save
```

创建src/Routes.js配置路由。

src/client/index.js使用BrowserRouter包裹住之前定义的Routes。

src/server/index.js使用StaticRouter来渲染Routes。context是StaticRouter做数据传递的，先写一个空对象让流程跑下去。

StaticRouter是不知道请求路径是什么的，因为他运行在服务器端，所以这是他不如BrowserRouter的地方，他需要在请求体重获取到路径传递给他, 这里需要将content写在请求里面。将location的值赋为req.path。

可以新建一个utils文件将通用方法抽离出来。

src/server/utils.js

```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom';
import Routes from '../Routes';
export const render = (req) => {
    const content = renderToString((
        <StaticRouter location={req.path} context={\{\}}>
            <Routes />
        </StaticRouter>
    ));
    return `
        <html>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
    `;
}
```

src/server/index.js，使用render函数。

```js
import express from 'express';
import { render } from './utils';

const app = express();
app.use(express.static('public'));

app.get('*', function(req, res) {
    res.send(render(req));
})
var server = app.listen(3000);
```

创建一个公共组件src/components/Header/index.js

```js
import React from 'react';

const Header = () => {
    return <div>header</div>
}

export default Header;
```

src/components/Home/index.js组件中引入Header组件。

```js
import React from 'react';
import Header from '../Header';

const Home = () => {
    return <div>
            <Header />
            Home
            <button onClick={() => { alert('click1')}>按钮</button>
        </div>
}

export default Home;
```

src/components/Login/index.js

```js
import React from 'react';
import Header from '../Header';

const Login = () => {
    return <div><Header />Login</div>
}

export default Login;
```

在Header中引入Link, 并且使用他跳转至Home和Login。

src/components/Header/index.js

```js
import React from 'react';
import { Link } from 'react-router-dom';

const Header = () => {
    return <div>
        <Link to="/">Home</Link>
        <br />
        <Link to="/login">Login</Link>
    </div>
}

export default Header;
```

在做页面同构的时候，服务器端渲染只会在第一次进入页面的时候进行，后面使用Link的跳转都是浏览器端的跳转，不会再去加载页面的资源文件。所以服务器端渲染不是每个页面都做服务器端渲染，而是访问的第一个页面具有服务端渲染的特性，其他的页面仍旧是React的路由机制。

## 5. Redux

如果想在项目中使用redux我需要在server端和client端都使用redux，react-redux可以方便开发者在react中去使用redux。redux-thunk是redux的一个中间件。

```s
npm install redux react-redux redux-thunk --save
```

src/client/index.js，引入createStore来创建store。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import Routes from '../Routes';
import { createStore } from 'redux';
import { Provider } from 'react-redux';

const reducer = (state = { name: 'yd'}, action) => {
    return state;
}
const store = createStore(reducer);

const App = () => {
    return (
        <Provider store={store}>
            <BrowserRouter>
                {Routes}
            </BrowserRouter>
        </Provider>
    )
}

ReactDOM.hydrate(<App />, document.getElementById('root'));
```

在src/components/Home/index.js组件中使用redux。

```js
import React from 'react';
import Header from '../Header';
import { connect } from 'react-redux';

const Home = (props) => {
    return <div>
        <Header>
        <div>{props.name}</div>
        <div>Home</div>
        <button onClick={() => { alert('click1'); }>按钮</button>
    </div>
}

const mapStatetoProps = state => ({
    name: state.name
});

export default connect(mapStatetoProps, null)(Home);
```

src/server/utils.js

```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom';
import Routes from '../Routes';
import { createStore } from 'redux';
import { Provider } from 'react-redux';

export const render = (req) => {

    const reducer = (state = { name: 'yd'}, action) => {
        return state;
    }
    const store = createStore(reducer);

    const content = renderToString((
        <Provider store={store}>
            <StaticRouter location={req.path} context={\{\}}>
                <Routes />
            </StaticRouter>
        </Provider>
    ));
    return `
        <html>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
    `;
}
```

使用redux的时候使用一些中间件，可以在src/server/utils.js演示。

```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom';
import Routes from '../Routes';
import { createStore, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';
import thunk from 'redux-thunk';

export const render = (req) => {

    const reducer = (state = { name: 'yd'}, action) => {
        return state;
    }
    const store = createStore(reducer, applyMiddleware(thunk));

    const content = renderToString((
        <Provider store={store}>
            <StaticRouter location={req.path} context={\{\}}>
                <Routes />
            </StaticRouter>
        </Provider>
    ));
    return `
        <html>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
    `;
}
```

src/client/index.js

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import Routes from '../Routes';
import { createStore, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';
import thunk from 'redux-thunk';

const reducer = (state = { name: 'yd'}, action) => {
    return state;
}
const store = createStore(reducer, applyMiddleware(thunk));

const App = () => {
    return (
        <Provider store={store}>
            <BrowserRouter>
                {Routes}
            </BrowserRouter>
        </Provider>
    )
}

ReactDOM.hydrate(<App />, document.getElementById('root'));
```

client和server中都会用到store，可以将他们抽离出来不要每个位置都写一遍。

注意，在render方法中每个用户访问都会使用store，但是在服务器上这个store只定义了一次，并不是每次调用render都会创建，所以共享了store，这样是不对的，每个用户都应该有自己的store。这里导出一个创建Store的方法，在使用的位置创建store。

src/store/index.js

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
const reducer = (state = { name: 'yd'}, action) => {
    return state;
}

const getStore = () => {
    return createStore(reducer, applyMiddleware(thunk));
}
export default getStore;
```

src/client/index.js也要加一下。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import Routes from '../Routes';
import getStore from '../store'; // 使用store
import { Provider } from 'react-redux';

const App = () => {
    return (
        <Provider store={getStore()}>
            <BrowserRouter>
                {Routes}
            </BrowserRouter>
        </Provider>
    )
}

ReactDOM.hydrate(<App />, document.getElementById('root'));
```

src/server/utils.js

```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter } from 'react-router-dom';
import Routes from '../Routes';
import getStore from '../store'; // 使用store
import { Provider } from 'react-redux';

export const render = (req) => {

    const content = renderToString((
        <Provider store={getStore()}>
            <StaticRouter location={req.path} context={\{\}}>
                <Routes />
            </StaticRouter>
        </Provider>
    ));
    return `
        <html>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
    `;
}
```

希望访问根目录的时候页面显示一个列表。修改首页src/components/Home/store/reducer.js初始化创建一些数据并且处理actions变化数据的变化。

```js
import { CHANGE_LIST } from './constants';
const defaultState = {
    newsList: []
}
export default (state = defaultState, action) => {
    switch (action.type) {
        case CHANGE_LIST:
            return {
                ...state,
                newsList: action.list
            };
        default:
            return state;
    }
}
```

在全局store中，对所有的reducer做组合。

src/store/index.js

```js
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import { reducer as homeReducer} from '../components/Home/store';

const reducer = combineReducers({
    home: homeReducer
});

const getStore = () => {
    return createStore(reducer, applyMiddleware(thunk));
}
export default getStore;
```

src/components/Home/store/index.js

```js
import reducer from './reducer';

export { reducer };
```

src/components/Home/index.js希望在这个文件中发一个请求去展示列表，这里改造成一个类式组件。借助dispatch的能力，在getHomeList中发送异步请求，dispatch需要使用action

```js
import React, { Component } from 'react';
import Header from '../Header';
import { connect } from 'react-redux';
import { getHomeList } from './store/actions';

class Home extends Component {

    getList() {
        const { list } = this.props;
        return this.props.list.map(item => <div key={item.id}>{item.title}</div>)
    }
    render() {
        return <div>
            <Header>
            <div>Home</div>
            {this.getList()}
            <button onClick={() => { alert('click1'); }>按钮</button>
        </div>
    }

    componentDidMount() {
        this.props.getHomeList();
    }
}

const mapStatetoProps = state => ({
    list: state.home.newsList
});

const mapDispatchToProps = dispatch => ({
    getHomeList() {
        dispatch(getHomeList());
    }
})

export default connect(mapStatetoProps, mapDispatchToProps)(Home);
```

定义函数，可以返回对象作为action也可以返回函数来做异步的操作，这是redux-thunk带来的能力。

src/components/Home/store/actions.js

```js
import axios from 'axios';
import { CHANGE_LIST } from './constants';

const changeList = (list) => {
    type: CHANGE_LIST,
    list
}

export const getHomeList = () => {
    return (dispatch) => {
        return axios.get('http://127.0.0.1:3000/getlist').then(res => {
            const list = res.data.data;
            dispatch(changeList(list));
        })
    }
}
```

src/components/Home/store/constants.js 存储常量。

```js
export const CHANGE_LIST = 'HOME/CHANGE_LIST';
```

这样redux的结构就创建好了，但是页面渲染的结构中并没有这段列表的结构。这是因为服务器端运行的时候componentDidMount并不会执行，所以列表是空的，用户看到的列表是客户端客户端运行的时候执行的。

在服务器端的时候也要执行componentDidMount获取到数据。将页面结构渲染出来。在Home组件中添加一个静态方法Home.loadData, 这个函数负责在服务端渲染之前把这个路由需要的数据提前加载好。

src/components/Home/index.js

```js
import React, { Component } from 'react';
import Header from '../Header';
import { connect } from 'react-redux';
import { getHomeList } from './store/actions';

class Home extends Component {

    getList() {
        const { list } = this.props;
        return this.props.list.map(item => <div key={item.id}>{item.title}</div>)
    }
    render() {
        return <div>
            <Header>
            <div>Home</div>
            {this.getList()}
            <button onClick={() => { alert('click1'); }>按钮</button>
        </div>
    }

    componentDidMount() {
        this.props.getHomeList();
    }
}

Home.loadData = (store) => {
    // 执行action，扩充store。
    return store.dispatch(getHomeList());
}

const mapStatetoProps = state => ({
    list: state.home.newsList
});

const mapDispatchToProps = dispatch => ({
    getHomeList() {
        dispatch(getHomeList());
    }
})

export default connect(mapStatetoProps, mapDispatchToProps)(Home);
```

componentDidMount在服务端是不会执行的，src/server/utils.js中拿到的store是空的，让store在渲染时拿到真实的数据就可以了。这里需要知道当前路由加载哪些组件的数据，需要改造一下路由配置，根据路由来判断加载的数据。loadData是加载组件之前执行的方法，这里写成Home.loadData, Login组件不需要加载任何数据，不定义就可以了。

src/Routes.js

```js
import React from 'react';
import Home from './components/Home';
import Login from './components/Login';

export default [
    {
        path: '/',
        component: Home,
        exact: true,
        key: 'home',
        loadData: Home.loadData
    },
    {
        path: '/login',
        component: Login,
        key: 'login',
        exact: true
    }
]
```

因为路由的结构改变了，所以这里使用Router.js的地方也要修改，Router是数组。

src/client/index.js注意需要使用div包裹一下所有Route，否则会报错。因为react-route-dom要求route成组出现。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter, Route } from 'react-router-dom';
import routes from '../Routes';
import getStore from '../store'; // 使用store
import { Provider } from 'react-redux';

const App = () => {
    return (
        <Provider store={getStore()}>
            <BrowserRouter>
                <div>
                    {
                        routes.map(route => (
                            <Route {...route} />
                        ))
                    }
                </div>
            </BrowserRouter>
        </Provider>
    )
}

ReactDOM.hydrate(<App />, document.getElementById('root'));
```

src/server/utils.js除了修改routes以外还要判断当前访问的路径是什么，然后将对应需要加载的数据提前放在store里面，需要借助matchRoute方法。他是react-router-config插件提供的。matchRoute可以匹配二级路由，react-router-dom自带的matchPath只能匹配一级路由。

render方法需要多接收一个res，用于返回给浏览器的send调用。因为请求函数是异步的，所以需要在回调结束后返回。

```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter, Route } from 'react-router-dom';
import routes from '../Routes';
import getStore from '../store'; // 使用store
import { Provider } from 'react-redux';
import { matchRoute } from 'react-router-config';

export const render = (req, res) => {
    const store = getStore();
    // 可以拿到store填充到store中。
    // 根据路由的路径向store里面添加数据，需要借助matchRoute，返回值是一个数组，里面是匹配到的每级路由。
    const matchedRoutes = matchRoute(routes, req,path);
    // 让matchRoutes里面所有的组件对应的loadData方法都执行一次
    // item.route.loadData返回的是一个promise等待promise执行完毕再向下，所以使用Promise.all，请求响应后返回给浏览器数据。
    const promises = [];
    matchedRoutes.forEach(item => {
        if (item.route.loadData) {
            promises.push(item.route.loadData(store));
        }
    });
    Promise.all(promises).then(() => {
        const content = renderToString((
            <Provider store={store}>
                <StaticRouter location={req.path} context={\{\}}>
                    <div>
                        {
                            routes.map(route => (
                                <Route {...route} />
                            ))
                        }
                    </div>
                </StaticRouter>
            </Provider>
        ));
        res.send(`
            <html>
                <body>
                    <div id="root">${content}</div>
                    <script src="/index.js"></script>
                </body>
            </html>
        `);
    })
}
```

src/server/index.js也需要修改一下。将req和res都传递进去，然后在render方法里面去返回响应。

```js
import express from 'express';
import { render } from './utils';

const app = express();
app.use(express.static('public'));

app.get('*', function(req, res) {
    render(req, res)
})
var server = app.listen(3000);
```

浏览器会自动请求favicon文件，造成代码重复执行，可以在public文件夹中加入这个图片解决该问题。这样访问浏览器就可以发现页面结构已经渲染出来了，并且是服务端渲染的而不是浏览器渲染的。

整理一下这些代码。src/server/index.js接收到用户请求的时候将store的创建移动到这里，保持render函数干净。

```js
import express from 'express';
import { matchRoute } from 'react-router-config';
import { render } from './utils';
import getStore from '../store'; // 使用store
import routes from '../Routes';

const app = express();
app.use(express.static('public'));

app.get('*', function(req, res) {
    const store = getStore();
    const matchedRoutes = matchRoute(routes, req,path);
    const promises = [];
    matchedRoutes.forEach(item => {
        if (item.route.loadData) {
            promises.push(item.route.loadData(store));
        }
    });
    Promise.all(promises).then(() => {
        res.send(render(store, routes, req)); 
    })
})
var server = app.listen(3000);
```

src/server/utils.js
```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter, Route } from 'react-router-dom';
import { Provider } from 'react-redux';

export const render = (store, routes, req) => {
    const content = renderToString((
        <Provider store={store}>
            <StaticRouter location={req.path} context={\{\}}>
                <div>
                    {
                        routes.map(route => (
                            <Route {...route} />
                        ))
                    }
                </div>
            </StaticRouter>
        </Provider>
    ));
    return `
        <html>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
    `;
}
```

总结一下服务器端渲染的流程。

当用户请求网页的时候首先服务端创建了一个空的store，然后根据请求路径和路由项做匹配，来判断用户当前访问的路径对应的加载项有哪些，所以matchedRoutes中放置的就是要展示的组件。循环matchedRoutes判断组件里面是否存在loadData,如果有就说明他需要加载一些数据，把loadData执行，将请求放在Promise的数组里面，等所有组件对应的Promose都执行完成之后就说明这一个路径要展示的组件依赖数据已经准备好了，结合展示好了的数据和路由最终生成一段html内容返回给用户。

不过这里还有一个问题，当加载页面之后页面会闪动一下，这是js执行的时候会清空页面再展示页面。因为客户端一开始store也是空的，是请求结束之后才有数据的。需要用到一个概念叫做脱水和注水，首先找到src/server/utils.js, 在渲染页面的时候可以在页面底部加一个script标签, 在这个标签里面写上渲染的数据。

src/server/utils.js

```jsx
import React from 'react';
import { renderToString } from 'react-dom/server';
import { StaticRouter, Route } from 'react-router-dom';
import { Provider } from 'react-redux';

export const render = (store, routes, req) => {
    const content = renderToString((
        <Provider store={store}>
            <StaticRouter location={req.path} context={\{\}}>
                <div>
                    {
                        routes.map(route => (
                            <Route {...route} />
                        ))
                    }
                </div>
            </StaticRouter>
        </Provider>
    ));
    return `
        <html>
            <body>
                <div id="root">${content}</div>
                <script>
                    window.context = {
                        state: ${JSON.stringfiy(store.getState())}
                    }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
    `;
}
```

打开src/store/index.js, 在里面新增一个方法getClientStore。

```js
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import { reducer as homeReducer} from '../components/Home/store';

const reducer = combineReducers({
    home: homeReducer
});

export const getStore = () => {
    return createStore(reducer, applyMiddleware(thunk));
}

export const getClientStore = () => {
    const defaultState = window.context.state;
    // defaultState作为默认值
    return createStore(reducer, defaultState, applyMiddleware(thunk));
}
```

src/server/index.js修改store获取方式

```js
import express from 'express';
import { matchRoute } from 'react-router-config';
import { render } from './utils';
import { getStore } from '../store'; // 使用store
import routes from '../Routes';

const app = express();
app.use(express.static('public'));

app.get('*', function(req, res) {
    const store = getStore();
    const matchedRoutes = matchRoute(routes, req,path);
    const promises = [];
    matchedRoutes.forEach(item => {
        if (item.route.loadData) {
            promises.push(item.route.loadData(store));
        }
    });
    Promise.all(promises).then(() => {
        res.send(render(store, routes, req)); 
    })
})
var server = app.listen(3000);
```

然后打开src/client/index.js这里的getStore换成getClientStore, 客户端的store要使用服务端给的store。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter, Route } from 'react-router-dom';
import routes from '../Routes';
import { getClientStore } from '../store'; // 