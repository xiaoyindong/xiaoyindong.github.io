## 1. 基本语法实现

定义```render```函数，接收```html```字符串，和```data```参数。

```js
const render = (ejs = '', data = {}) => {

}
```

事例模板字符串如下:

```ejs
<body>
    <div><%= name %></div>
    <div><%= age %></div>
</body>
```

可以使用正则将```<%= name %>```匹配出来，只保留```name```。这里借助```ES6```的模板字符串。将```name```用```${}```包裹起来。

```props```中第```2```个值就是匹配到的变量。直接```props[1]```替换。

```js
[
  '<%= name %>',
  ' name ',
  16,
  '<body>\n    <div><%= name %></div>\n    <div><%= age %></div>\n</body>'
]
```

```js
const render = (ejs = '', data = {}) => {
    const html = ejs.replace(/<%=(.*?)%>/g, (...props) => {
        return '${' + props[1] + '}';
        // return data[props[1].trim()];
    });
}
```

## 2. Function函数

这里得到的```html```是一个模板字符串。可以通过```Function```将字符串编程可执行的函数。当然这里也可以使用```eval```，随你。

```js
<body>
    <div>${ name }</div>
    <div>${ age }</div>
</body>
```

```Function```是一个构造函数，实例化后返回一个真正的函数，构造函数的最后一个参数是函数体的字符串，前面的参数都为形式参数。比如这里传入形参name，函数体通过```console.log```打印一句话。

```js
const func = new Function('name', 'console.log("我是通过Function构建的函数，我叫：" + name)');
// 执行函数，传入参数
func('yindong'); // 我是通过Function构建的函数，我叫：yindong
```

利用```Function```的能力可以将```html```模板字符串执行返回。函数字符串编写```return```，返回一个拼装好的模板字符串、

```js
const getHtml = (html, data) => {
    const func = new Function('data', `return \`${html}\`;`);
    return func(data);
    // return eval(`((data) => {  return \`${html}\`; })(data)`)
}

const render = (ejs = '', data = {}) => {
    const html = ejs.replace(/<%=(.*?)%>/g, (...props) => {
        return '${' + props[1] + '}';
    });
    return getHtml(html, data);
}
```

## 3 with

这里```render```函数中```props[1]```的实际上是变量名称，也就是```name```和```age```，可以替换成```data[props[1].trim()]```，不过这样写会有一些问题，偷个懒利用```with```代码块的特性。

```with```语句用于扩展一个语句的作用域链。换句人话来说就是在```with```语句中使用的变量都会先在```with```中寻找，找不到才会向上寻找。

比如这里定义一个```age```数字和```data```对象，```data```中包含一个```name```字符串。```with```包裹的代码块中输出的```name```会先在```data```中寻找，```age```在```data```中并不存在，则会向上寻找。当然这个特性也是一个```with```不推荐使用的原因，因为不确定```with```语句中出现的变量是否是```data```中。

```js
const age = 18;
const data = {
    name: 'yindong'
}

with(data) {
    console.log(name);
    console.log(age);
}
```

这里使用```with```改造一下```getHtml```函数。函数体用```with```包裹起来，```data```就是传入的参数```data```，这样```with```体中的所有使用的变量都从```data```中查找了。

```js
const getHtml = (html, data) => {
    const func = new Function('data', `with(data) { return \`${html}\`; }`);
    return func(data);
    // return eval(`((data) => { with(data) { return \`${html}\`; } })(data)`)
}

const render = (ejs = '', data = {}) => {
    // 优化一下代码，直接用$1替代props[1];
    // const html = ejs.replace(/<%=(.*?)%>/g, (...props) => {
    //     return '${' + props[1] + '}';
    // });
    const html = ejs.replace(/<%=(.*?)%>/gi, '${$1}');
    return getHtml(html, data);
}
```

这样就可以打印出真是的```html```了。

```html
<body>
    <div>yindong</div>
    <div>18</div>
</body>
```

## 4. ejs语句

这里扩展一下```ejs```，加上一个```arr.join```语句。

```ejs
<body>
    <div><%= name %></div>
    <div><%= age %></div>
    <div><%= arr.join('--') %></div>
</body>
```

```js
const data = {
    name: "yindong",
    age: 18,
    arr: [1, 2, 3, 4]
}

const html = fs.readFileSync('./html.ejs', 'utf-8');

const getHtml = (html, data) => {
    const func = new Function('data', ` with(data) { return \`${html}\`; }`);
    return func(data);
}

const render = (ejs = '', data = {}) => {
    const html = html = ejs.replace(/<%=(.*?)%>/gi, '${$1}');
    return getHtml(html, data);
}

const result = render(html, data);

console.log(result);
```

可以发现```ejs```也是可以正常编译的。因为模板字符串支持```arr.join```语法，输出：

```html
<body>
    <div>yindong</div>
    <div>18</div>
    <div>1--2--3--4</div>
</body>
```

如果```ejs```中包含```forEach```语句，就比较复杂了。此时```render```函数就无法正常解析。

```ejs
<body>
    <div><%= name %></div>
    <div><%= age %></div>
    <% arr.forEach((item) => {%>
        <div><%= item %></div>
    <%})%>
</body>
```

这里分两步来处理。仔细观察可以发现，使用变量值得方式存在```=```号，而语句是没有```=```号的。可以对```ejs```字符串进行第一步处理，将```<%=```变量替换成对应的变量，也就是原本的```render```函数代码不变。

```js
const render = (ejs = '', data = {}) => {
    const html = ejs.replace(/<%=(.*?)%>/gi, '${$1}');
    console.log(html);
}
```

```ejs
<body>
    <div>${ name }</div>
    <div>${ age }</div>
    <% arr.forEach((item) => {%>
        <div>${ item }</div>
    <%})%>
</body>
```

第二步比较绕一点，可以将上面的字符串处理成多个字符串拼接。简单举例，将```a```加上```arr.forEach```的结果再加上```c```转换为，```str```存储```a```，再拼接```arr.forEach```每项结果，再拼接```c```。这样就可以获得正确的字符串了。

```js
// 原始字符串
retrun `
    a
    <% arr.forEach((item) => {%>
        item
    <%})%>
    c
`
// 拼接后的
let str;
str = `a`;

arr.forEach((item) => {
    str += item;
});

str += c;

return str;
```

在第一步的结果上使用```/<%(.*?)%>/g```正则匹配出```<%%>```中间的内容，也就是第二步。

```js
const render = (ejs = '', data = {}) => {
    // 第一步
    let html = ejs.replace(/<%=(.*?)%>/gi, '${$1}');
    // 第二步
    html = html.replace(/<%(.*?)%>/g, (...props) => {
        return '`\r\n' + props[1] + '\r\n str += `';
    });
    console.log(html);
}
```

替换后得到的字符串长成这个样子。

```js
<body>
    <div>${ name }</div>
    <div>${ age }</div>
    `
 arr.forEach((item) => {
 str += `
        <div>${ item }</div>
    `
})
 str += `
</body>
```

添加换行会更容易看一些。可以发现，第一部分是缺少首部\`的字符串，第二部分是用```str```存储了```forEach```循环内容的完整```js```部分，并且可执行。第三部分是缺少尾部\`的字符串。

```js
// 第一部分
<body>
    <div>${ name }</div>
    <div>${ age }</div>
    `

// 第二部分
 arr.forEach((item) => {
 str += `
        <div>${ item }</div>
    `
})

// 第三部分
 str += `
</body>
```

处理一下将字符串补齐，在第一部分添加let str = \`，这样就是一个完整的字串了，第二部分不需要处理，会再第一部分基础上拼接上第二部分的执行结果，第三部分需要在结尾出拼接\`; return str; 也就是补齐尾部的模板字符串，并且通过```return```返回str完整字符串。

```js
// 第一部分
let str = `<body>
    <div>${ name }</div>
    <div>${ age }</div>
    `

// 第二部分
 arr.forEach((item) => {
 str += `
        <div>${ item }</div>
    `
})

// 第三部分
 str += `
</body>
`;

return str;
```

这部分逻辑可以在getHtml函数中添加，首先在with中定义str用于存储第一部分的字符串，尾部通过return返回str字符串。

```js
const getHtml = (html, data) => {
    const func = new Function('data', ` with(data) { let str = \`${html}\`; return str; }`);
    return func(data);
}
```

这样就可以实现执行ejs语句了。

```js
const data = {
    name: "yindong",
    age: 18,
    arr: [1, 2, 3, 4],
    html: '<div>html</div>',
    escape: '<div>escape</div>'
}

const html = fs.readFileSync('./html.ejs', 'utf-8');

const getHtml = (html, data) => {
    const func = new Function('data', ` with(data) { var str = \`${html}\`; return str; }`);
    return func(data);
}

const render = (ejs = '', data = {}) => {
    // 替换所有变量
    let html = ejs.replace(/<%=(.*?)%>/gi, '${$1}');
    // 拼接字符串
    html = html.replace(/<%(.*?)%>/g, (...props) => {
        return '`\r\n' + props[1] + '\r\n str += `';
    });
    return getHtml(html, data);
}

const result = render(html, data);

console.log(result);
```

输出结果

```ejs
<body>
    <div>yindong</div>
    <div>18</div>

        <div>1</div>

        <div>2</div>

        <div>3</div>

        <div>4</div>

</body>
```

## 5. 标签转义

```<%=```会对传入的```html```进行转义，这里编写一个```escapeHTML```转义函数。

```js
const escapeHTML = (str) => {
    if (typeof str === 'string') {
        return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/ /g, "&nbsp;").replace(/"/g, "&#34;").replace(/'/g, "&#39;");
    } else {
        return str;
    }
}
```

变量替换的时候使用```escapeHTML```函数处理变量。这里通过```\s*```去掉空格。为了避免命名冲突，这里将```escapeHTML```改造成自执行函数，函数参数为```$1```变量名。

```js
const render = (ejs = '', data = {}) => {
    // 替换转移变量
    // let html = ejs.replace(/<%=\s*(.*?)\s*%>/gi, '${escapeHTML($1)}');
    let html = ejs.replace(/<%=\s*(.*?)\s*%>/gi, `\${
        ((str) => {
            if (typeof str === 'string') {
                return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/ /g, "&nbsp;").replace(/"/g, "&#34;").replace(/'/g, "&#39;");
            } else {
                return str;
            }
        })($1)
    }`);
    // 拼接字符串
    html = html.replace(/<%(.*?)%>/g, (...props) => {
        return '`\r\n' + props[1] + '\r\n str += `';
    });
    return getHtml(html, data);
}
```

```getHtml```函数不变。

```js
const getHtml = (html, data) => {
    const func = new Function('data', `with(data) { var str = \`${html}\`; return str; }`);
    return func(data);
}
```

```<%-```会保留原本格式输出，只需要再加一条不使用```escapeHTML```函数处理的就可以了。

```js
const render = (ejs = '', data = {}) => {
    // 替换转义变量
    let html = ejs.replace(/<%=\s*(.*?)\s*%>/gi, '${escapeHTML($1)}');
    // 替换其余变量
    html = html.replace(/<%-(.*?)%>/gi, '${$1}');
    // 拼接字符串
    html = html.replace(/<%(.*?)%>/g, (...props) => {
        return '`\r\n' + props[1] + '\r\n str += `';
    });
    return getHtml(html, data, escapeHTML);
}
```

输出样式

```js
<body>
    <div>yindong</div>
    <div>18</div>

        <div>1</div>

        <div>2</div>

        <div>3</div>

        <div>4</div>

    <div>&lt;div&gt;escapeHTML&lt;/div&gt;</div>
</body>
```

至此一个简单的```ejs```模板解释器就写完了。
