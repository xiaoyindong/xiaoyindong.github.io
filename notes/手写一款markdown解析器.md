## 1. 准备工作

首先编写```getHtml```函数，传入```markdown```文本字符串，这里使用```fs```读取```markdown```文件内容，返回值是转换过后的字符串。

```js
const fs = require('fs');

const source = fs.readFileSync('./test.md', 'utf-8');

const getHtml = (source) => {
    // 处理标题
    return source;
}

const result = getHtml(source);

console.log(result);
```

主要设计正则表达式和String.prototype.replace方法，replace接收的第一个参数可以是正则，第二个参数如果是函数那么返回值就是所替换的内容。

## 2. 处理图片&超链接

图片和超链接的语法很像，```![图片](url)```，```[超链接](url)```，使用正则匹配同时需要排除`。props会获取正则中的```$```，```$1```，```$2```。也就是匹配的字符整体，第一个括号内容，第二个括号内容。比如这里```props[0]```就是匹配到的完整内容，第四个参数```props[3]```是```[]```中的```alt```，第五个参数```props[4]```是链接地址。

```js
const imageora = (source) => {
    return source.replace(/(`?)(!?)\[(.*)\]\((.+)\)/gi, (...props) => {
        switch (props[0].trim()[0]) {
            case '!': return `<a href="${props[4]}" alt="${props[3]}">${props[3]}</a>`;
            case '[': return `<img src="${props[4]}" alt="${props[3]}"/>`;
            default: return props[0];
        }
    });
}

const getHtml = (source) => {
    source = imageora(source);
    return source;
}
```

## 3. 处理blockquote

这里使用```\x20```匹配空格。如果匹配到内容，将文本```props[3]```放在blockquote标签返回就行了。

```js
const block = (source) => {
    return source.replace(/(.*)(`?)\>\x20+(.+)/gi, (...props) => {
        switch (props[0].trim()[0]) {
            case '>': return `<blockquote>${props[3]}</blockquote>`;
            default: return props[0];
        }
    });
}
```

## 4. 处理标题

匹配必须以```#```开头，并且```#```的数量不能超过6，因为h6是最大的了，没有h7，最后```props[2]```是```#```后跟随的文本。

```js
const formatTitle = (source) => {
    return source.replace(/(.*#+)\x20?(.*)/g, (...props) => {
        switch (props[0][0]) {
            case '#': if (props[1].length <= 6) {
                return `<h${props[1].length}>${props[2].trim()}</h${props[1].length}>`;
            };
            default: return props[0];
        }
    })
}
```

## 5. 处理字体

写的开始复杂了

```js
const formatFont = (source) => {
    // 处理 ~ 包裹的文本
    source = source.replace(/([`\\]*\~{2})(.*?)\~{2}/g, (...props) => {
        switch (props[0].trim()[0]) {
            case '~': return `<del>${props[2]}</del>`;;
            default: return props[0];
        }
    });
    // 处理 * - 表示的换行
    source = source.replace(/([`\\]*)[* -]{3,}\n/g, (...props) => {
        switch (props[0].trim()[0]) {
            case '*': ;
            case '-': return `<hr />`;
            default: return props[0];
        }
    })
    // 处理***表示的加粗或者倾斜。
    source = source.replace(/([`\\]*\*{1,3})(.*?)(\*{1,3})/g, (...props) => {
        switch (props[0].trim()[0]) {
            case '*': if (props[1] === props[3]) {
                if (props[1].length === 1) {
                    return `<em>${props[2]}</em>`;;
                } else if (props[1].length === 2) {
                    return `<strong>${props[2]}</strong>`;;
                } else if (props[1].length === 3) {
                    return `<strong><em>${props[2]}</em></strong>`;;
                }
            };
            default: return props[0];
        }
    });
    return source;
}
```

## 6. 处理代码块

使用正则匹配使用\`包裹的代码块，```props[1]```是开头\`的数量，```props[5]```是结尾\`的数量，必须相等才生效。

```js
const pre = (source) => {
    source = source.replace(/([\\`]+)(\w+(\n))?([^!`]*?)(`+)/g, (...props) => {
        switch (props[0].trim()[0]) {
            case '`': if (props[1] === props[5]) {
                return `<pre>${props[3] || ''}${props[4]}</pre>`;
            };
            default: return props[0];
        }
    });
    return source;
}
```

## 7. 处理列表

这里只是处理了```ul```无序列表，写的同样很麻烦。主要我的思路是真复杂。而且bug肯定也不少。先匹配-+*加上空格，然后根据这一行前面的空格熟替换为ul。这样每一行都保证被ulli包裹。

第二步判断相邻ul之间相差的个数，如果相等则表示应该是同一个ul的li，替换掉```</ul><ul>```为空，如果后一个ul大于前一个ul，则表示后面有退格，新生成一个```<ul>```包裹退格后的li，如果是最后一个ul则补齐前面所有的```</ul>```。

```js
const list = (source) => {
    source = source.replace(/.*?[\x20\t]*([\-\+\*]{1})\x20(.*)/g, (...props) => {
        if (/^[\t\x20\-\+\*]/.test(props[0])) {
            return props[0].replace(/([\t\x20]*)[\-\+\*]\x20(.*)/g, (...props) => {
                const len = props[1].length || '';
                return `<ul${len}><li>${props[2]}</li></ul${len}>`;
            })
        } else {
            return props[0];
        }
    });
    const set = new Set();
    source = source.replace(/<\/ul(\d*)>(\n<ul(\d*)>)?/g, (...props) => {
        set.add(props[1]);
        if (props[1] == props[3]) {
            return '';
        } else if (props[1] < props[3]) {
            return '<ul>';
        } else {
            const arr = [...set];
            const end = arr.indexOf(props[1]);
            let start = arr.indexOf(props[3]);
            if (start > 0) {
                return '</ul>'.repeat(end - start);
            } else {
                return '</ul>'.repeat(end + 1);
            }            
        }
    });
    return source.replace(/<(\/?)ul(\d*)>/g, '<$1ul>');
}
```

## 8. 处理表格

真的没耐心了。。。

```js
const table = (source) => {
    source = source.replace(/\|.*\|\n\|\s*-+\s*\|.*\|\n/g, (...props) => {
        let str = '<table><tr>';
        const data = props[0].split(/\n/)[0].split('|');
        for (let i = 1; i < data.length - 1; i++) {
            str += `<th>${data[i].trim()}</th>`
        }
        str += '<tr></table>';
        return str;
    });
    return formatTd(source);
}

const formatTd = (source) => {
    source = source.replace(/<\/table>\|.*\|\n/g, (...props) => {
        let str = '<tr>';
        const data = props[0].split('|');
        for (let i = 1; i < data.length - 1; i++) {
            str += `<td>${data[i].trim()}</td>`
        }
        str += '<tr></table>';
        return str;
    });
    if (source.includes('</table>|')) {
        return formatTd(source);
    }
    return source;
}
```

## 9. 调用方法

```js
const getHtml = (source) => {
    source = imageora(source);
    source = block(source);
    source = formatTitle(source);
    source = formatFont(source);
    source = pre(source);
    source = list(source);
    source = table(source);
    return source;
}

const result = getHtml(source);

console.log(result);
```

正则有点入门了，接着还是去看看开源的markdown解析器怎么实现的吧。

水一水，生活真快乐。