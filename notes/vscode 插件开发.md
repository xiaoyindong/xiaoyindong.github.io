## 1. 概述

本文并非将```VS Code```插件的每个```API```都介绍一遍，仅会介绍几种主流的插件类型，尤其是一些工作中可能会用得着的插件类型。

```VS Code```是通过```Electron```来实现的跨平台，而```Electron```则是基于```Chromium```和```Node.js```，比如```VS Code```的界面是通过```Chromium```进行渲染的。同时```VS Code```是多进程架构，当```VS Code```第一次启动时会创建一个主进程(```main process```)，然后每个窗口都会创建一个渲染进程(```Renderer Process```)。同时```VS Code```会为每个窗口创建一个进程专门来执行插件也就是```Extension Host```。

除了这三个主要的进程以外，还有两种特殊的进程第一种是调试进程，```VS Code```为调试器专门创建了```Debug Adapter```进程，渲染进程会通过```VS Code```Debug Protocol```跟```Debug Adapter```进程通讯。

对于插件作者而言，无需关心```VS Code```的架构，在书写```VS Code```插件的时候，只需知道插件就是一个```Node.js```应用，其次在这个```Node.js```应用中可以直接访问```VS Code```的```API```，通过这些```API```来操作```VS Code```， 并不需要知道插件进程是怎么跟渲染进程通讯的。

最后每当打开一个窗口时，```VS Code```会为这个窗口创建插件进程，并且按需要激活插件。也就是说，同一时间代码有可能被运行多次。

## 2. 创建插件

```VS Code```官方自提供了基于```NPM```的工具帮助创建和维护插件。首先需要的是```yeoman```脚手架工具。通过```yeoman```可以快速创建代码模板。

```s
npm insall -g yeoman
```

安装```VS Code```的模板

```s
npm insall -g generator-code
```

创建```VS Code```插件模板了

```s
yo code myextension
```

这里有七个插件模板。前两个是通过编程来提供插件功能，可以选择```TypeScript```或者```JavaScript```，第三个是主题插件，可以将创建的主题分享给其他人，第四个是语言支持，也就是语法高亮、语言定义，第五个是代码片段的分享，第六个是快捷键，第七个是对多个插件进行组合分享。

暂时可以选择第二项```New Extension (JavaScript)```，接下来会依次被提示输入插件的名字、介绍、想要用哪个账号发布、是否要打开```type check```以及是否要使用```git```等。

输入全部问题后，脚本就会自动地创建文件，安装需要的```dependencies```。全部结束后，可以运行下面的脚本打开这个插件的代码。

```s
cd myextension
code .
```

```VS Code```的脚手架，默认创建了不少的文件，```package.json```里记录了```Node.js```应用的信息同时插件的信息也会被记录在这个文件内。```extension.js```文件是当前插件的全部代码。```.vscode```脚手架工具已经提供了调试配置、任务配置等，有了它们，就不用自己花时间书写了。

```extension.js```的内容在删除了所有的注释后。

```js
cons vscode = require('vscode');
function activate(context) {
    console.log('Congratulations, your extension "myextension" is now active!');
    let disposable = vscode.commands.regiserCommand('extension.sayHello', function () {
        vscode.window.showInformationMessage('Hello World!'); 
    });
    context.subscriptions.push(disposable); 
}

exports.activate = activate;

function deactivate() {
}

exports.deactivate = deactivate;
```

首先引用了```vscode```库。通过引用这个库，就能够使用```VS Code```的插件```API```了。第二创建了```activate```函数并且将其输出。```VS Code```的插件进程在激活插件时，就是调用这个被输出(```export```)的函数。也就是说，这个函数就是这个插件的入口。相对应的是```deactivate```函数，当禁用这个插件或者关闭```VS Code```时，这个函数就会被调用了。

activate函数如下。

```js
function activate(context) {
    console.log('Congratulations, your extension "myextension" is now active!');
    let disposable = vscode.commands.regiserCommand('extension.sayHello', function () {
        vscode.window.showInformationMessage('Hello World!'); 
    });
    context.subscriptions.push(disposable); 
}`
```

这个函数首先输出了```log```，表示插件已经被成功激活了。接着，使用```vscode.commands.regiserCommand```注册```extension.sayHello```命令，这个命令的实现是```regiserCommand的```第二个参数，通过调用```vscode.window.showInformationMessage```，在界面上调出一个提示框内容是```Hello World!```。

光有```extension.js```插件是无法运行的。```VS Code```会根据条件来激活插件，激活条件写在```package.json```中。

```json
{
    "name": "myextension",
    "displayName": "myextension",
    "description": "my extension",
    "version": "0.0.1",
    "publisher": "rebornix",
    "engines": {
        "vscode": "^1.29.0"
    },
    "categories": [
        "Other"
    ],
    "activationEvents": [
        "onCommand:extension.sayHello"
    ],
    "main": "./extension",
    "contributes": {
        "commands": [
            {
                "command": "extension.sayHello",
                "title": "Hello World"
            }
        ]
    },
    "scripts": {
        "posinsall": "node ./node_modules/vscode/bin/insall",
        "tes": "node ./node_modules/vscode/bin/tes"
    },
    "devDependencies": {
        "typescript": "^2.6.1",
        "vscode": "^1.1.21",
        "eslint": "^4.11.0",
        "@types/node": "^8.10.25",
        "@types/mocha": "^2.2.42"
    }
}
```

上面这个文件，跟普通的```npm```的```package.json```只有三处不同。第一处是```engines```。它指定了运行这个插件需要的```VS Code```版本。比如```^1.29.0```就是说明，要安装运行这个插件必须要使用```VS Code```1.29```及以上版本。

```s
"vscode": "^1.29.0"
```

```activationEvents```这个属性指定了什么情况下这个插件应该被加载并且激活。在例子里激活条件是当用户想要运行```extension.sayHello```命令时就激活这个插件。这个机制能够保证，需要使用插件的时候才被激活，尽可能地保证性能和内存使用的合理性。

```s
"activationEvents": [ 
    "onCommand:extension.sayHello"
]
```

```contributes```属性指定了插件给```VS Code```添加了```command```，```command```的```id```是```extension.sayHello```， 跟```extension.js```中写的一样。命令的名字叫做```Hello World```。如果不写这个属性```VS Code```是不会把这个命令注册到命令面板中的，也就没法找到这个命令并且执行。

```json
"contributes": { 
    "commands": [
        {
            "command": "extension.sayHello", 
            "title": "Hello World"
        } 
    ]
},
```

## 3. 运行插件

```VS Code```插件代码脚手架提供了```launch.json```，只需要按下```F5```即可启动代码。代码启动后，```VS Code```会打开一个新的窗口，这个窗口中运行着本地书写的代码。此时打开命令面板，搜索H```ello World```并且执行。

这个插件只有在```Hello World```命令被执行时才会被激活。可以将窗口关掉，然后在```activate```函数中加上断点，重新试一次。当```Hello World```命令被执行时，首先被唤起的是```activate```函数，然后是```showInformationMessage```。

以上就是```VS Code```的插件架构以及插件示例里的几个重要文件的作用，并且成功地将一个插件运行起来并且激活、执行命令。可以修改代码，```VS Code```为```JavaScript```代码提供了不错的智能提示，可以轻松找到```VS Code```有哪些可用的插件```API```。

## 4. 编写编辑器命令

首先修改代码示例中```extension.js```的内容现在如下。

```js
cons vscode = require('vscode');

function activate(context) {
    console.log('Congratulations, your extension "myextension" is now active!');
    let disposable = vscode.commands.regiserCommand('extension.sayHello', function () {
        vscode.window.showInformationMessage('Hello World!'); 
    });
    let editor = vscode.window.activeTextEditor;
        if (!editor) { return;
    }
   context.subscriptions.push(disposable); 
}

exports.activate = activate;

function deactivate() {
}

exports.deactivate = deactivate;
```
### 1. 访问编辑器

首先要获取的就是当前工作区内，用户正在使用的编辑器。

```js
let editor = vscode.window.activeTextEditor;
```

有了编辑器，就能获取非常多的信息了。```editor```变量并非一定总是有效的，比如用户现在并没有打开任何文件，编辑器是空的，那么此时```editor```的值就是```undefned```。所以， 在正式使用```editor```之前，要判断一下```editor```是否为```undefned```，是的话就结束命令的运行。

```js
if (!editor) {
    return;
}
```

接下来，可以输入```editor.```。```document```是当前编辑器中的文档内容，```edit```用于修改编辑器中的内容，```revealRange```用于将某段代码滚动到当前窗口中，```selection```是当前编辑器内的主光标，```selections```是当前编辑器中的所有光标，第一个光标是主光标，后面的则是用户创建出来的多光标，```setDecorations```设置编辑器装饰器，这个命令可以将光标左、右 两侧的字母位置调换。不过如果将多个字符选中，运行这个命令并不能将它们反转。下面，来看看如何实现字符串反转。

首先，要读取的信息就是当前的文档信息和主光标的信息。

```js
let document = editor.document;
let selection = editor.selection;
```

有了这两个信息，读取光标选中的内容就简单了。

```js
let text = document.getText(selection);
```

这里使用就是```document.getText```获取某段代码。接这将这段文本进行反转，可以写一个非常简单的版本，将字符串分割成字母数组，然后反转，最后重新组合成字符串。

```js
let result = text.split('').reverse().join('');
```

最后一步操作是将原来编辑器内的文本进行替换了。此时要用到```edit```这个```API```。值得注意的是，这个```API```的第一参数是一个```callback```，```callback```的参数是```editBuilder```，也就是真正用于修改代码的对象。```editBuilder```有以下几个```API```。```delete```，```insert```，```replace```，```setEndOfLine```。这里要使用的是```replace```。

```js
editBuilder.replace(selection, result);
```

将原先```selection```里的内容，替换成新的```result```即可。将代码调试运行起来，选中代码```path```，运行```Hello World```命令，```path```就被替换为了```htap```。

绝大多数的编辑器命令的工作方式，基本上跟上面的示例如出一辙。一共分为三部分。首先读取文档中的内容需要使用的```API```是```selection```、```selections```、```getText```等。其次对这些内容进行二次加工，这部分就是```business logic```了。最后修改编辑器内的内容可以使用```edit```来修改文本，也可以直接修改```editor.selection```和```editor.selections```来改变光标的位置。

不过，如果要书写一个没有```bug```且性能出色的编辑器命令，可就没那么简单了。比如上面的示例里面， 没有对多光标进行支持，反转字符串也是很暴力的，而这一部分，才是插件真正体现差距的地方。

### 2. 快捷键

上面介绍了如何书写一个命令，但是这只完成了工作的一半，剩下的一半则是为这个命令绑定一个快捷键。要完成快捷键的绑定，需要在```package.json```中的```contributes```片段添加一段新的配置。

```json
"contributes": {
    "commands": [
        {
            "command": "extension.sayHello",
            "title": "Hello World"
        }
    ],
    "keybindings": [
        {
            "key": "ctrl+t",
            "command": "extension.sayHello",
            "when": "editorTextFocus"
        }
    ]
},
```

在```contributes```添加了新的字段```keybindings```，它的值是一个数组，里面是所有的快捷键设置。给```extension.sayHello```命令，绑定```ctrl + t```，同时只有当```editorTextFocus```为真时才会激活这个快捷键。运行插件后可以直接使用```ctrl + t```来反转字符串了。

```keybindings```配置，还可以给```VS Code```已经存在的命令重新指定快捷键。

### 3. 分享快捷键

```VS Code```有一套插件叫做```keymap```。可以在插件市场找到所有的```keymap```。这里面除了```Vim```比较特殊以外，其他的```keymap```基本上都是使用```keybindings```来重新指定快捷键的。如果你看```Notpad++```的源代码时，会发现这个插件连```javascript```文件都没有，只有一个长达```258```行的```package.json```。通过这套```keybindings```，可以在```VS Code```中使用```Notepad++```的快捷键。

```json
"activationEvents": ["*"]
```

```Notepad++ keymap```的```activationEvents```是```*```，意思是不管什么条件，永远都会激活这个插件。对于```keymap```这样需要覆盖绝大多数命令的插件而言，将其设置为```*```无伤大雅。不过，如果你的插件被使用的频率并不算高，还是需要精心设计```activationEvents```，关于可以使用的```activationEvents```，还请查看```VS Code```文档。

## 5. 分享代码和主题

代码片段可以通过```yeoman```脚手架创建一个代码片段分享的插件模板，脚本依然是```yo code```。这一次选择```Code Snippet```。

可以提供```TextMate```或者```Sublime```的代码片段文件，```VS Code```脚手架工具会自动将它们转成```VS Code```支持的格式。如果并不是要从已有的代码片段转换过来也没关系，可以直接按下回车创建新的代码片段文件。在输入了插件名称、```id```、发布者名称等之后，脚手架又提问，这个代码片段是为哪个语言准备的。每个语言都会拥有一个自己的代码片段。

输入语言后插件模板就被创建出来了。打开新创建出来的文件夹，这个模板比上面的```JavaScript```的插件模板还简单，没有了```extension.js```、```eslint```配置等文件，而是多出了一个```snippets/snippets.json```文件。

这里要着重介绍的是```package.json```里```contributes```的变化。

```json
"contributes": {
    "snippets": [
        {
            "language": "javascript",
            "path": "./snippets/snippets.json"
        }
    ]
}
```

```contributes```中的值不再是```commands```，而是```snippets```，它里面指定一个```snippet```文件的相对地址。可以将代码片段放入到```snippets/snippets.json```文件中去，然后就可以通过插件分享给其他人了。

主题的分享就更简单了，依然是通过脚手架来创建模板。首模板类型是```New Color Theme```，接着脚手架询问是否要倒入已经存在的主题文件。可以使用```TextMate```、```Atom```或者```Sublime```的主题文件，因为大家使用的主题引擎都是一样的。当然从零开始创建一个主题文件也非常简单，选择```No, start fresh```。

创建的最后一个问题是想要创建的主题是深色的，浅色的还是高对比度的，选择后```VS Code```会根据基础主题默认提供一部分颜色，然后就可以基于它再进行拓展。

插件被创建好后，会发现它跟代码片段的模板很接近，只不过多了一个```themes/mytheme- color-theme.json```文件。这个文件就是对编辑器内代码以及工作区的颜色设置。当基于某个现成的主题修改配色后，可以将添加的配置```workbench.colorCustomizations```和```editor.tokenColorCustomizations```拷贝进这个文件中。不过还有一个更简单的方式打开命令面板，搜索```使用当前设置生成主题```并执行。

这个生成出来的文件，可以当作插件进行分享的主题文件。总结来说，要创建一个颜色主题可以先在个人设置中修改工作区或者编辑器内的主题，然后使用命令```使用当前设置生成主题```生成主题文件，并为这个主题文件添加```name```名字，将这个文件分享成插件，最后在```package.json```的```contributes```部分注册这个主题文件。

```json
"contributes": {
    "themes": [
        {
            "label": "mytheme",
            "uiTheme": "vs-dark",
            "path": "./themes/mytheme-color-theme.json"
        }
    ]
}
```

配置里```label```是主题的名字，```uiTheme```是基础主题，```path```是主题文件的相对地址。

## 6. 自定义语言

在```VS Code```中自定义语言支持并不只是```TypeScript```、```Rust```等语言开发者的特权，任何人都可以借助```VS Code```的插件定义和```API```实现它们，甚至不需要书写```Language Server```，也能达成一样的效果。

首先要做的是创建一个语言支持相关的插件模板。这一次选择```New Language Support```。接着，我脚手架工具需要提供```tmLanguage```。

```tmLanguage```是```TextMate```创造的定义语言语法的文件，```Sublime```、```Atom```以及```VS Code```都是继承自此。如果其他的编辑器已经支持了某个语言，大可将其直接引入。这也是为什么```VS Code```插件```API```一经发布，很快大部分在```Textmate```、```Sublime```上支持的语言就在```VS Code```上得到了支持，都拥有不错的语法高亮。

关于如何书写```tmLanguage```，```TextMate```官方有一个简短但翔实的文档```Language Grammars — TextMate 1.x Manual```。精髓是通过书写正则表达式，对代码进行搜索匹配，最后给每个代码片段标注上类型(```token type```)。暂时可以不提供```tmLanguage```，直接按下回车。

在输入完插件名、描述信息和发布者信息后，```yeoman```提了三个跟语言相关的问题。

第一个是```id```，```id```是这门语言的唯一标识，最重要的是不要和已经存在的主流语言产生冲突。第二个和第三个是语言的名字和后缀。

最后脚手架工具问了一个跟```tmLanguage```有关的问题，就是指定这个语言的```scope name```。暂时叫它```source.mylanguage```好了。回答了全部的问题后就可以创建出模板了。

通过```package.json```中```contributes```的值，可以发现这个插件注册两个信息。一个是```tmLanguage```，用于语法高亮。另一个是```languages```也就是```mylanguage```这个语言的信息。它的信息包含以下几点。

```id```是语言独一无二的标识，```alias```是语言的名字、别称，```extensions```是这个语言相关文件的后缀名，设置了这个值之后，当打开一个后缀为```.mylanguage```的文件，```VS Code```就知道把它识别成什么语言了，```confgurations```是这个语言的配置信息所在文件的相对地址。

前面三个是使用```yeoman```创建插件时提供的，最后这个```language-confguration.json```是干什么的呢?

```json
{
    "comments": {
        // symbol used for single line comment. Remove this entry if your language does not support line comments
        "lineComment": "//",
        // symbols used for sart and end a block comment. Remove this entry if your language does not support block comments
        "blockComment": [
            "/*",
            "*/"
        ]
    },
    // symbols used as brackets 
    "brackets": [
        [
            "{","}"
        ],
        [
            "[","]"
        ],
        [
            "(",")"
        ]
    ],
    // symbols that are auto closed when typing 
    "autoClosingPairs": [
        [
            "{","}"
        ],
        [
            "[","]"
        ],
        [
            "(",")"
        ],
        [
            "\"","\""
        ],
        [
            "'","'"
        ]
    ],
    // symbols that that can be used to surround a selection 
    "surroundingPairs": [
        [
            "{","}"
        ],
        [
            "[","]"
        ],
        [
            "(",")"
        ],
        [
            "\"","\""
        ],
        [
            "'","'"
        ]
    ]
}
```

这个设置已经提供了一些语言相关信息的模板了。

第一个是代码注释```comments```的格式。```VS Code```允许提供两种不同格式的注释—单行注释和多行注释。提供这两个信息后，在编辑器里使用```Toggle Comment```命令时，```VS Code```会根据选择的内容的行数来决定使用哪个格式的注释。

```brackets```是指这个语言支持的括号类型。要注意的是，这里输入的括号类型，只支持单个字符，比如```Ruby```的```def/end```就不能在这里使用了。

```autoClosingPairs```是括号自动配对，比如输入了```{```，编辑器会自动补上```}```。```autoClosingPairs```的书写格式有两种。第一种是类似于```[“{”, “}”]```的数组形式，其中第一个元素就是开括号，第二个则是关括号。第二种书写格式则是输入一个对象。

```json
{ "open": "/**", "close": " */", "notIn": ["sring"] }
```

通过```open```和```close```属性来指定开关括号。这里的```括号```则是一个相对抽象的概念，并不一定真的是一对括号，比如在上面的例子里，```open```属性的值是```/```，而 close 属性则是```*/```，那么当输入```/```后， 编辑器就会立刻替你补上```*/```。可以通过这个来实现注释的自动补全。

与此同时，这种写法还支持一个新的属性```notIn```。意思是在哪种代码里不进行自动补全。 比如在写注释的时候，可能就不希望将引号自动补全，不然的话，当输入```it’s```这种单词时，输入完单引号后，如果编辑器自作主张帮忙输入了另一个单引号，可就多此一举了。此时可以通 过```notIn```巧妙地避开了这一功能。

```json
{ "open": "'", "close": "'", "notIn": ["sring", "comment"] }
```

```notIn```可以填入的值有```string```和```comment```。

```surroundingPairs```里的字符对适用于对选中的字符串进行自动包裹。比如当选中了```abc```这三个单词，然后按下双引号，此时，```VS Code```就会把```abc```直接变成```“abc”```，而不是将```abc```替换成双引号。这个功能就叫做```auto surround```。可以通过这个属性来决定语言里面，哪些字符对可以直接```auto surround```。


除了上面这几个外，在```language-confguration.json```里，可以使用的属性有```WordPattern```、```Folding```和```Indentation```。这三个属性，分别定义了在这门语言中，一个单词长什么样、可被折叠的代码段的开头和结尾各长什么样以及如何根据一行的内容来控制缩进。

```json
"folding": {
    "markers": {
        "sart": "^\\s*//#region",
        "end": "^\\s*//#endregion"
    }
},
"wordPattern": "(-?\\d*\\.\\d\\w*)|([^\\`\\~\\!\\@\\#\\%\\^\\&\\*\\(\\)\\-\\=\\+\\[\\ {\\]\\}\\\\\\|\\;\\:\\'\\\"\\,\\.\\<\\>\\/\\?\\s]+)",
"indentationRules": {
    "increaseIndentPattern": "^\\s*((begin|class| (private|protected)\\s+def|def|else|elsif|ensure|for|if|module|rescue|unless|until|when|while|case)| ([^#]*\\sdo\\b)|([^#]*=\\s*(case|if|unless)))\\b([^#\\{;]|(\"|'|\/).*\\4)*(#.*)?$",
    "decreaseIndentPattern": "^\\s*(}\\]]?\\s*(#|$)|\\.[a-zA-Z_]\\w*\\b)| (end|rescue|ensure|else|elsif|when)\\b)"
}
```

这三个属性的键都还是很好理解的，不过要注意，它们的值都是正则表达式。```VS Code```在读取了这些正则表达式后，就会拿它们对代码进行匹配。比如用鼠标双击代码，编辑器就会尝试着选中当前的单词，而这个单词就可以由```wordPattern```来决定。再比如默认情况下，```VS Code```会认为空格符并不是单词的一部分，但是有些语言则不然，空格也能成为合法的变量名称，这种时候，就可以在```wordPattern```中添加空格符以达到这一目的。

```folding```设置里的```start```和```end```告诉编辑器哪一行代码是可折叠代码的开始和结束，而这些语法规则就是来自```folding```里的```start```和```end```。

```indentationRules```这条规则则是来自于```TextMate```，感兴趣的同学可以阅读```TextMate```相关的文档```Appendix — TextMate 1.x Manual```。

## 7. Language Server Protocol

```language-confguration.json```和```tmLanguage```，一个控制了语言的```Folding```、括号、单词等基础信息，另一个控制了语言的语法```tokenization```规则。有了这两个文件，就可以获得语法高亮和 基于文本的各类编辑器功能了。

不过，如果想在编辑器内提供一定的自动补全或者代码跳转，还要写一点代码。在```VS Code```的插件```API```里包含了```Language Server Protocol```的```API```。也就是说，通过书写简单的```JavaScript```代码，可以为```VS Code```提供语言服务，不理解```Language Server Protocol```也没关系，因为每个```API```都可以单独拿出来使用。

下面来实现一个自动补全的插件。可以沿用之前的插件模板，这个插件只有一个代码文件```extension.js```。

不过这一次不是注册一个命令，而是要注册一个自动补全提供者(```provider```)。先输入```vscode.languages.```，然后看看自动补全都提示了什么。

```languages```下能够注册各种不同的语言功能，比如自动补全 (```CompletionItemProvider```)、代码跳转(```DefnitionProvider```)、格式化 (```DocumentFormattingEditProvider```)等等。一个完整的```Language Server```，其实就将这些方法通通实现，只不过```Language Server```是运行在另一个独立的进程里。而使用插件```API```时，选择只实现某些```API```。这里选择```CompletionItemProvider```，需要提供三个参数。

第一个是```DocumentSelector```。控制在哪些文件中提供自动补全。类型可以是字符串，也可以是```DocumentFilter```。先试用```plaintext```。

第二个是```CompletionItemProvider```了。这个对象至少要拥有下面这个函数属性。

```js
provideCompletionItems(document: TextDocument, position: Position, token: CancellationToken, context: CompletionContext): ProviderResult<CompletionItem[] | CompletionLis>;
```

```VS Code```在调用这个函数时，会提供当前的文档```document```， 光标所在的位置```position```，用于监测用户取消建议操作的```CancellationToken```，以及最后一个当前补全项的上下文```context```。

有了```document```和```position```，就能够分析全文代码并且推测当前光标处可能的建议选项了。返回值可以是```CompletionItem[]```或者```CompletionLis```。

```js
export class CompletionItem {
    label: sring;
    kind?: CompletionItemKind;
    detail?: sring;
    documentation?: sring | MarkdownString;
    sortText?: sring;
    flterText?: sring;
    preselect?: boolean;
    insertText?: sring | SnippetString;
    range?: Range;
    commitCharacters?: sring[];
    keepWhitespace?: boolean;
    additionalTextEdits?: TextEdit[];
    command?: Command;
    consructor(label: sring, kind?: CompletionItemKind);
}

export class CompletionLis {
    isIncomplete?: boolean;
    items: CompletionItem[];
    consructor(items?: CompletionItem[], isIncomplete?: boolean);
}
```

虽然```CompletionItem```的属性非常多，但实际上除了```label```以外，其他都是```optional```的。```label```就是建议项的名字，用于建议列表里的显示。不过，除了```label```，还必须告诉```VS Code```，当用户从建议列表里选择了这个建议项之后，按下回车，该如何修改光标处的代码。这时候就要使用```insertText```和```range```这两个属性了，它们决定了将哪段代码替换成什么新的文本。

```regiserCompletionItemProvider```的第三个属性，是```triggerCharacters```决定了当用户按下哪个字符之后，编辑器就应该立刻询问```CompletionItemProvider```以提供自动补全建议。比如只在用户按下```.```和```(```时触发自动补全，那么这里要输入的就是```[‘.’, ‘(’]```。

```extension.js```

```js
cons vscode = require('vscode');
function activate(context) { 
    vscode.languages.regiserCompletionItemProvider('plaintext', {
        provideCompletionItems: (document, position) => {
            return [
                {
                    label: 'mySuggesion', 
                    insertText: 'mySuggesion'
                } 
            ]
        }
    }, ['.'])
}

exports.activate = activate;

function deactivate() {

}
exports.deactivate = deactivate;
```

需要将```package.json```里的```activationEvents```改成```*```，

```js
"activationEvents": [ "*"],
```

```F5```启动插件运行后，打开一个纯文本，输入```this.```，当输入```.```之后，立刻看到了建议列表。然后按下回车，插件提供的建议就被插入到编辑器里了。

使用上面的样例代码，当将光标移动到```vscode.languages```上时，按下```F12```，```VS Code```就立刻跳转到```VS Code```插件```API```的```typings```文件里。这个文件是所有插件```API```的定义。如果是在```registerCompletionItemProvider```上按```F12```，则是直接跳转到这个```API```的定义处。可以通过跳转定义命令(```F12```)，一个个查询每个类型。

```VS Code```的插件```API```太多了，有了```typings```文件，不离开```VS Code```，就能够了解到它们各自是怎么使用的。

## 8. Decorations 装饰器

首先，使用```JavaScript```插件模板。```extension.js```文件内容如下。

```js
cons vscode = require('vscode');
function activate(context) { 
    vscode.commands.regiserCommand('extension.sayHello', () => {
        let decorationType = vscode.window.createTextEditorDecorationType({ backgroundColor: '#fff' });
        let editor = vscode.window.activeTextEditor; editor.setDecorations(decorationType, [new vscode.Range(0, 0, 0, 1)]);
    }); 
}

exports.activate = activate;

function deactivate() {

}

exports.deactivate = deactivate;
```

注册一个名为```extension.sayHello```的命令，创建```DecorationType```。

```js
let decorationType = vscode.window.createTextEditorDecorationType({ backgroundColor: '#fff'})
```

使用```vscode.window.createTextEditorDecorationType```传入参数对象，对象里添加了属性```backgroundColor```用于定义背景色的。

获取当前的编辑器对象```editor```。使用```editor```上的```setDecorations```方法，并且传入两个参数，第一个是我创建的```DecorationType```，第二个是代码范围```Range```。通过这个```API```，在代码片段```Range```上使用```decorationType```所代表的装饰器，也就是将背景色调整成```#fff```。

运行插件，无论是```Pigment```、```Rainbow Brackets```，还是```GitLens```，它们修改编辑器内代码颜色和背景色的逻辑跟上面的这一小段代码基本一致。只不过```VS Code```的```Decoration API```，有非常多不同的属性可以设置，这也让```Decoration```效果千差万别。

如果在```createTextEditorDecorationType```运行```F12```，就能够跳转到```createTextEditorDecorationType```函数的定义处，函数的参数类型是```DecorationRenderOptions```。```createTextEditorDecorationType```的结构如下。

```js
export interface DecorationRenderOptions extends ThemableDecorationRenderOptions { 
    isWholeLine?: boolean;
    rangeBehavior?: DecorationRangeBehavior; 
    overviewRulerLane?: OverviewRulerLane; 
    light?: ThemableDecorationRenderOptions; 
    dark?: ThemableDecorationRenderOptions;
}
```

```createTextEditorDecorationType```继承自```ThemableDecorationRenderOptions```，不过它多加了几个属性。比如是否将```decoration```运用在整行代码上，是否要将颜色渲染在滚动条上等。不过比较重要的两个属性其实是```light```和```dark```。

```light```和```dark```的类型，都是```ThemableDecorationRenderOptions```，只要设置了这两个值，```VS Code```就会根据当前的主题是深色还是浅色，决定是加载```light```还是```dark```的值，如果这两个值没有设置的话，那么就会查看```DecorationRenderOptions```上的其他属性。```ThemableDecorationRenderOptions```有如下属性。

```js
export interface ThemableDecorationRenderOptions { 
    backgroundColor?: sring | ThemeColor; outline?: sring;
    outlineColor?: sring | ThemeColor; outlineStyle?: sring;
    outlineWidth?: sring;
    border?: sring;
    borderColor?: sring | ThemeColor; borderRadius?: sring; borderSpacing?: sring; borderStyle?: sring; borderWidth?: sring;
    fontStyle?: sring;
    fontWeight?: sring;
    textDecoration?: sring;
    cursor?: sring;
    color?: sring | ThemeColor;
    opacity?: sring;
    letterSpacing?: sring;
    gutterIconPath?: sring | Uri;
    gutterIconSize?: sring;
    overviewRulerColor?: sring | ThemeColor;
    before?: ThemableDecorationAttachmentRenderOptions; after?: ThemableDecorationAttachmentRenderOptions;
}
```

可以给这些属性归归类。

### 1. Color 

代码颜色和背景相关的属性。

```js
    backgroundColor?: sring | ThemeColor; 
    color?: sring | ThemeColor; 
    overviewRulerColor?: sring | ThemeColor;
```

除了使用类似于```#fff```这样的字符串以外，还可以使用```ThemeColor```，也就是颜色主题(```themes```) 里的颜色定义，比如。

```js
new vscode.ThemeColor('editorWarning.foreground')
```

这样一来，当切换主题的时候，颜色会随之改变。

### 2. Border

第二类是```Border```边框。

```js
border?: sring;
borderColor?: sring | ThemeColor; 
borderRadius?: sring; 
borderSpacing?: sring; 
borderStyle?: sring; 
borderWidth?: sring;
```

比如对上面的样例代码略作修改。

```js
vscode.commands.regiserCommand('extension.sayHello', () => {
    let decorationType = vscode.window.createTextEditorDecorationType({
        border: '1px solid red;' 
    });
    let editor = vscode.window.activeTextEditor;
    editor.setDecorations(decorationType, [new vscode.Range(0, 0, 0, 1)]); 
});
```

然后运行代码，就能够给代码块添加边框了。

### 3. Outline

在```CSS```中```Outline```(轮廓)是用于在```Border```边框的周围画一条线，以突出元素。我们同样可以使用下面这些配置，来分别控制```Outline```的各个属性。

```js
    outline?: sring;
    outlineColor?: sring | ThemeColor; 
    outlineStyle?: sring; 
    outlineWidth?: sring;
```

比如，我们将样例代码修改成如下:

```js
vscode.commands.regiserCommand('extension.sayHello', () => {
    let decorationType = vscode.window.createTextEditorDecorationType({
        outline: '#00FF00 dotted' 
    });
    let editor = vscode.window.activeTextEditor; editor.setDecorations(decorationType, [new vscode.Range(1, 1, 1, 4)]);
})
```

为了更好地看到效果，这里把```range```修改成了```new vscode.Range(1, 1, 1, 4)```。

### 4. Font

可以通过```fontStyle```和```fontWeight```来控制字体，也可以通过```opacity```来控制透明度，或者使用```letterSpacing```控制文字之间的间距。

```js
    fontStyle?: sring; 
    fontWeight?: sring; 
    opacity?: sring; 
    letterSpacing?: sring;
```

比如代码修改为。

```js
vscode.commands.regiserCommand('extension.sayHello', () => {
    let decorationType = vscode.window.createTextEditorDecorationType({ 
        fontStyle: 'italic',
        letterSpacing: '3px'
    });
    let editor = vscode.window.activeTextEditor; editor.setDecorations(decorationType, [new vscode.Range(1, 1, 1, 4)]);
});
```

### 5. 其他

除此之外，还可以通过```gutterIconPath```和```gutterIconSize```在行号旁边添加图标。

```js
    gutterIconPath?: sring | Uri; 
    gutterIconSize?: sring;
```

比如说```VS Code```中的断点，就可以通过这个属性在插件中实现。另外，也可以通过```before```和```after```在某个代码的前面或者后面创建```decorations```。

```js
before?: ThemableDecorationAttachmentRenderOptions; 
after?: ThemableDecorationAttachmentRenderOptions;
```

```CSS```文件里颜色前的```Color Decorator```小方格，就是使用```before```属性来实现。

```DecorationRenderOptions```里的大部分属性，其实就是```CSS```里的各种属性，如果知道如何使用```CSS```来对元素布局的话，那么使用这些```API```就难不倒你。不过最难的还是想象力，如何活用这套```API```，将重要的信息呈现给用户，而又不会打扰到用户的正常体验，这就体现功力了。

比如之前介绍过```Import Cost```插件，这个插件就巧妙地将每个```javascript```模块的大小，渲染在了这一行代码的最后，十分显眼，但又不会影响到查看代码。

```GitLens```插件也有类似的设计，它可以把当前这行是谁写的从```Git```中读取出来，然后渲染在本行代码的最后，同时这些信息的颜色比较浅，从而不会喧宾夺主，当需要的时候也可以轻松迅速地查看代码的修改信息。文章最开始介绍的```Rainbow Brackets```等也都是对```Decoration API```的活用。

## 9. 工作台 API

第一类是通过插件```API```在```VS Code```中调出对话框向用户询问问题，或者弹出信息提示以警示用户。二者的本质是一致的，都是跟用户进行信息的交互，以完成进一步的操作。

首先来看信息提示，这个```API```在之前的例子里已经使用过了。

比如下面的这段示例代码，注册了```extension.sayHello```命令，通过```vscode.window.showInformationMessage```输出提示```Hello World```。

```js
vscode.commands.regiserCommand('extension.sayHello', () => { 
    vscode.window.showInformationMessage('Hello World');
});
```

至于对话框可以使用的```API```有两种```showWarningMessage```和```showErrorMessage```。使用方法都是一样的，不过呈现效果会不同，用以体现```Information、Warning```和```Error```不同的重要程度。

除了给用户展示信息以外，这三个```API```还允许和用户进行简单的交互。先来看```showInformationMessage```的定义。

```js
/**
* Show an information message to users. Optionally provide an array of items which will be presented as
* clickable buttons. *
* @param message The message to show.
* @param items A set of items that will be rendered as actions in the message.
* @return A thenable that resolves to the selected item or `undefned` when being dismissed. 
*/

export function showInformationMessage(message: sring, ...items: sring[]): Thenable<sring | undefned>;
```

除了传入消息```message```，还可以传入一个数组，这个数组里的字符串，都会被渲染成按钮，当用户按下这些按钮，就能够收到反馈了。

```js
vscode.commands.regiserCommand('extension.sayHello', () => { 
    vscode.window.showInformationMessage('Hello World', 'Yes', 'No').then(value => {
        vscode.window.showInformationMessage('User press ' + value); 
    })
});
```

给用户提供了```Yes```和```No```两个选项，当用户选择其中之一后，弹出一个新的信息框，显示用户点击了哪个。

```QuickPick```通过```vscode.window.showQuickPick```函数，给用户提供一系列选项，根据用户选择的选项进行下一步操作。这里给用户提供了三个选项```first```、```second```、```third```，然后将用户的选择以信息的方式弹出。

```js
vscode.window.showQuickPick(['first', 'second', 'third']).then(value => { 
    vscode.window.showInformationMessage('User choose ' + value);
})
```

```showQuickPick```的第一个参数除了可以是一个字符串数组以外，还可以提供其他不同的类型比如```Promise```，这个参数可以为最终解析值为字符串数组的```Promise```。有了这个类型，就能够异步地获取选项列表，等这个列表解析出来了再提供给用户，而用户则会在界面上看到滚动条。

另外，数组也可以是```QuickPickItem```对象数组。

```js
export interface QuickPickItem {
/**
* A human readable sring which is rendered prominent.
*/
    label: sring; 
    description?: sring; 
    detail?: sring; 
    picked?: boolean;
    alwaysShow?: boolean;
}
```

上面的示例里面使用的数组也可以用```QuickPickItem```来替代，只需要使用```QuickPickItem```的```label```属性，然后```label```里的值就会被渲染在列表中。

除了```label```还可以通过```description```或者```detail```来提供更多的信息。比如说使用下面的```QuickPickItem```数组。

```js
vscode.commands.regiserCommand('extension.sayHello', () => { 
    vscode.window.showQuickPick([{
        label: 'frs',
        description: 'frs item',
        detail: 'frs item details'
    },{
        label: 'second', 
        description: 'second item', 
        detail: 'second item details'
    }]).then(value => {
        vscode.window.showInformationMessage('User choose ' + value.label);
    }) 
});
```

至于```picked```属性就非常好理解了。默认情况下列表里的第一个选项会被选中。如果希望默认选中其他项的话，将它的```picked```属性改为```true```就好了。```alwaysShow```这个属性则是使用于列表很长的情况，如果列表非常长，```VS Code```不得不渲染出滚动条时，通过将某些项的```alwaysShow```属性改为```true```，这个选项就会一直出现在列表中，而不会受滚动条的影响。

整体来讲实现插件的过程中，很多命令或者操作的流程、信息，并不是完全确定的，往往需要用户来提供更多的信息，并且由用户来做出最终的决定。这个时候通过信息提示和```QuickPick```将选择权交还给了用户。

但是一定要注意的是信息提示和```QuickPick```都是会打扰用户的正常工作的。所以在使用这类```API```的时候一定要慎重，不然用户可能就会卸载我们的插件了。

## 10. 面板 Panel

第二类就是面板里的信息了。默认情况下，面板中有以下```4```个组件：问题面板，调试面板，输出面板，终端面板。这里面除了调试面板是由调试插件控制的以外，其他的三个，都是可以通过普通的插件```API```来完成的。这里面属问题面板和输出面板使用最为频繁。

### 1. 问题面板

在书写代码时```VS Code```的各类插件会把代码中出现的错误信息提供给问题面板。然后用户就可以通过问题面板，快速地查询问题并且进行代码的跳转。问题面板相关的```API```存在于```vscode.languages```的```namespace```下。要给问题面板提供相关的信息，使用的```API```是```createDiagnosticCollection```。

```js
export namespace languages { 
/**
* Create a diagnosics collection.
*
* @param name The name of the collection. * @return A new diagnosic collection.
*/
export function createDiagnosicCollection(name?: sring): DiagnosicCollection; }
```

通过```vscode.languages.createDiagnosticCollection```创建出来的对象，将是跟```VS Code```问题面板通讯的中介。下面使用如下的代码样例进行说明。

```js
vscode.commands.regiserCommand('extension.sayHello', () => {
    let collection = vscode.languages.createDiagnosicCollection('myextension');
    let uri = vscode.window.activeTextEditor.document.uri;
    collection.set(uri, [
        {
            range: new vscode.Range(0, 0, 0, 1), message: 'We found an error'
        } 
    ]);
});
```

在```collection```对象创建出来后，就要往这个```collection```里塞数据，这里使用的```API```是```set(uri: Uri, diagnosics: Diagnosic[] | undefned): void;```。

```set```函数提供两个参数第一个是文档的地址```Uri```，样例代码里使用了```vscode.window.activeTextEditor.document.uri```，也就是当前编辑器里的文档的地址```Uri```。第二个是在这个文档里发现的所有问题，每个问题的类型必须是```Diagnostic```。

```js
export class Diagnosic { 
/**
* The range to which this diagnosic applies. 
*/
    range: Range; /**
    * The human-readable message.
    */
    message: sring; /**
    * The severity, default is error.
    */
    severity: DiagnosicSeverity;

    source?: sring;
    code?: sring | number;
    relatedInformation?: DiagnosicRelatedInformation[];
    tags?: DiagnosicTag[];
    consructor(range: Range, message: sring, severity?: DiagnosicSeverity);
}
```

```Diagnostic```对象必须要提供的两个属性是```range```和```message```，也就是问题所在的位置和问题相关的信息。还可以给```Diagnostic```对象提供诸如```severity```问题的程度、```source```问题的来源等。

代码运行起来后在编辑器里执行```Hello World```命令，可以看到第一行第一列代码下出现了波浪线，同时问题面板里也多出了一个条目，点击它就能够跳转到编辑器中。

### 2. 输出面板

输出面板提供内容的```API```要更简单一些。首先要创建一个```OutputChannel```。只要提供一个名字即可。接着就可以往这个对象中添加输出日志了。有了这两行代码，就可以运行了。

```js
let channel = vscode.window.createOutputChannel('MyExtension');

channel.appendLine('Hello World');
```

通过上面的代码可以发现，输出面板下拉框中现在出现了一个新的选项，叫做```MyExtension```也就是创建的```OutputChannel```。接着使用```channel.appendLine```输出的信息，就会被放在输出面板中。这套```API```非常像```console.log()```，唯一不同的是这套```API```将内容输出到了输出面板中。

这部分总体来说就是问题面板的使用，跟语言服务结合到一起会很好，比如```Linting```信息、编译错误信息，甚至错别字检查信息，都可以塞到问题面板中。不过要注意，问题面板里的内容，意味着需要用户去修改代码。所以一些无关紧要的信息就不要放到这里面了。

而输出面板，完全可以把它当```log```日志来使用。大部分时间用户不需要去关心它，不过当用户遇到问题了，如果能够通过输出日志里的信息获得帮助，那么输出面板的目的就达到了。

### 3. 视图 TreeView

这一套```API```的最初需求是来自于```Visual Studio```用户，```Visual Studio```中，可以在视图中看到项目、测试、云管理等，但是```VS Code```当时并没有```API```可以实现这种定制。于是```TreeView```应运而生，通过实现这套```API```，任何插件都可以实现类似于资源管理器的树形结构。

```TreeView```虽然是用于创建视图中树形结构的，但是它跟```VS Code```的其他 API 非常类似，都是给```VS Code```提供数据，然后```VS Code```来进行渲染。创建```TreeView```的```API```也非常简单。

```js
export namespace window {
    export function regiserTreeDataProvider<T>(viewId: sring, treeDataProvider: TreeDataProvider<T>): Disposable;
}
```

```registerTreeDataProvider```一共有两个参数第一个是```TreeView```的名字，第二个是```TreeView```的数据来源```Data Provider```。

```js
export interface TreeDataProvider<T> { 
    onDidChangeTreeData?: Event<T | undefned | null>; 
    getTreeItem(element: T): TreeItem | Thenable<TreeItem>; 
    getChildren(element?: T): ProviderResult<T[]>; 
    getParent?(element: T): ProviderResult<T>;
}
```

```Data Provider```上只有两个属性是必须的。第一个是```getTreeItem```，通过这个函数，```VS Code```就知道该怎么渲染某个树节点了。第二个是```getChildren```，返回一个树节点的所有子节点的数据。```TreeItem```是每个树节点的数据结构。

```js
export class TreeItem {
    label?: sring;
    id?: sring;
    iconPath?: sring | Uri | { light: sring | Uri; dark: sring | Uri } | ThemeIcon; 
    resourceUri?: Uri;
    tooltip?: sring | undefned;
    /**
    * The command that should be executed when the tree item is selected.
    */
    command?: Command;
    /**
    * TreeItemCollapsibleState of the tree item. 
    */
    collapsibleState?: TreeItemCollapsibleState;
    contextValue?: sring;
    consructor(label: sring, collapsibleState?: TreeItemCollapsibleState); 
    consructor(resourceUri: Uri, collapsibleState?: TreeItemCollapsibleState);
}
```

```TreeItem```有两种创建方式第一种是提供```label```，也就是一个字符串，```VS Code```会把这个字符串渲染在树形结构中，第二种是提供```resourceUri```也就是一个资源地址，```VS Code```则会像资源管理器里渲染文件和文件夹一样渲染这个节点。

```iconPath```属性是用于控制树节点前的图标的。如果自己通过```TreeView```来实现一个资源管理器，就可以使用```iconPath```来为不同的文件类型指定不同的图标。```tooltip```属性是当把鼠标移动到某个节点上等待片刻，```VS Code```就会显示出这个节点对应的```tooltip```文字。```collapsibleState```用于控制这个树节点是应该展开还是折叠。当然，如果这个节点没有子节点的话，这个属性就用不着了。```command```属性如果存在的话，当点击这个树节点时，这个属性所指定的命令就会被执行了。

了解了以上几个属性就能够实现一个简易的```TreeView```了。

```js
       
vscode.window.regiserTreeDataProvider('myextension', { 
    getChildren: (element) => {
        if (element) { 
            return null;
        }
        return ['first', 'second', 'third']; 
    },
    getTreeItem: (element) => { 
        return {
            label: element,
            tooltip: 'my ' + element + ' item' 
        }
    } 
})
```

上面的这段代码注册了一个名为```myextension```的```TreeView```，这个```TreeView```只有一层节点，分别是```first```、```second```、```third```。将这段代码放入```extension.js```中时，运行插件会发现```VS Code```的视图里找不到这个名为```myextension```的```TreeView```。

```js
cons vscode = require('vscode');

function activate(context) { 
    vscode.window.regiserTreeDataProvider('myextension', {
        getChildren: (element) => { 
            if (element) {
                return null; 
            }
            return ['first', 'second', 'third']; 
        },
        getTreeItem: (element) => { 
            return {
                label: element,
                tooltip: 'my ' + element + ' item' 
            }
        } 
    })
}

exports.activate = activate;

function deactivate() {
}
exports.deactivate = deactivate;
```
   
这是因为要想将这个 TreeView 成功地注册到```VS Code```中，需要在```package.json```的```contributes```部分添加```TreeView```的申明。修改后的```package.json```的```contributes```部分如下。

```json
{
    "contributes": {
        "views": {
            "explorer": [
                {
                    "id": "myextension",
                    "name": "My Extension"
                }
            ]
        }
    }
}
```

这段```contributes```是说，把```myextension```这个```TreeView```注册到资源管理器中。除了将```TreeView```注册到资源管理器```Explorer```下以外，也可以将它注册到版本管理视图中，对应的```contributes```如下。

```json
"contributes": {
    "views": {
        "scm": [
            {
                "id": "myextension",
                "name": "My Extension"
            }
        ]
    }
}
```

代码运行起来后，就能够在版本管理视图中看到这个```TreeView```了。

简言之，```VS Code```的```TreeView```使用了```Data Provider```的模式，插件提供数据，而```VS Code```负责渲染。至于数据长什么样、树形结构里的层级关系如何，这个就属于```Business Logic```了，需要开发者自己发挥想象力。比如说```GitHub Pull Request```插件，用树形结构来展示所有的```Pull Requests```和每个```PR```里的代码改动，```NPM Explorer```则将所有的```NPM```脚本展示在树形结构中。

除此之外```VS Code```还有很多别的有趣的工作台相关的```API```，比如可以使用```WebView```来生成任意的编辑器内容，可以使用```FileSystemProvider```或者```TextDocumentContentProvider```来为```VS Code```提供类似于本地文件的文本内容。虽然它们很小众也更高级，但是使用的方法，跟上面提到的几种并没有什么区别，建议你通过```VS Code```的```typings```文件找寻你想要使用的```API```多多尝试。

## 11. 维护和发布

```VS Code```的插件```API```的发布流程首先是发出提议```Proposal```，看看社区的反馈如何。这一类```API```会出现在```vscode.proposed.d.ts```文件中，而稳定版本的```API```则是在```vscode.d.ts```里。一个```API```进入```proposed```状态并不需要什么流程，但是要进入```stable```的话，就要经过整个团队的```review```了。基本上一个```API```要发布到```stable```中，需满足以下几个条件。

首先，插件```API```不会将```UI```直接暴露给插件。```VS Code```的界面 (也就是```DOM```)的渲染完全由```VS Code```控制，插件```API```可以做的，就是将```UI```上的渲染逻辑翻译成```Data Provider```的形式，插件提供内容，```VS Code```负责渲染。

其次，如果一个```API```的运行时间可能比较长，那么这个```API```应该支持```Promise```，并且可以取消(也就是```vscode.d.ts```里常看到的```Cancellation Token```)。

最后，插件```API```能够正确地处理对象的生命周期。```VS Code```使用了```Dispose```模式，大部分```VS Code```插件```API```生成的对象，都会拥有一个```dispose```函数属性，然后运行这个函数就可以将这个对象销毁。

基于上面的这些```API```设计原则，也能够得出一个好的插件实现应该有如下特性。

对于长时间运行的任务，如果用户选择取消，那么插件应该能够终止任务。 插件能够及时地删除不再使用的对象，以及正确的时候```dispose VS Code```生成的对象，减少内存的使用。插件在给```VS Code```插件```API```提供数据的同时，能够做到增量更新，尽可能地减少```VS Code```重新渲染组件。只有做到上面这些，才能尽可能地保证插件的性能。插件提供的功能是一方面，但是如果性能出众的话，就真的是一个好插件了。

### 1. Node.js 模块使用

```VS Code```插件其实就是一个```Node.js```应用。那么如何管理```Node.js```的```dependencies```也是插件应该关心的。在使用第三方的```Node.js```模块时，要注意以下几点。

第一，很多简单的功能，其实可以自己实现，过多地使用第三方模块，会导致代码量不必要地增大。代码量增大，就相应地减慢了插件的下载和更新。同时插件被激活时，需要加载各个```Node.js```模块，模块越多，速度也就越慢。所以使用模块要克制。第二，如果可以的话，借助 webpack 对插件进行打包，并且开启```treeshaking```，把没有使用的代码删除掉。第三，对于性能要求比较高的应用，可以考虑使用```Node.js```的```Native Module```或者```Web Assembly```。最新版本的```VS Code```里已经支持了```Node.js```新的```Native Module API (N-API)```和```Web Assembly```了。不过这两者之间也各有优劣。

在```NAPI```之前，大家都在使用```NAN```来管理```Node.js Native Module```，但是一旦```VS Code```升级了```Electron```，导致```Node.js```版本发生变化，所有的```Native Module```就不能工作了。```NAPI```的出现，解决了这个问题，再也不用担心```Electron```升级的问题了。但是```NAPI```也并没有解决发布的问题，依然得为每个不同的平台(```Windows，macOS，Linux```)分别编译```Native Module```，比较麻烦。

相比于```Native Module```，使用```Web Assembly```就要好很多，因为```Web Assembly```天然就是跨平台的。但是它也有缺点，就是无法访问系统```API```，如果代码必须要访问到一些原生的```API```，可能还是得用```Native Module```。

以上的重点依然是性能。对于大部分插件而言```business logic```都不是特别复杂，而性能往往就是区分度，如果能够借助```Native Module、Web Assembly```或者```Webpack```等打包工具，给插件代码提速，就非常给力了。

### 2. 发布

插件的最终发布跟插件```API```相比简单很多。只需创建一个```Visual Studio Online```账户，然后使用```vsce```这个```npm```包就能发布了。现在```Marketplace```更是允许直接在后台发布，而无需使用命令行。关于更多的细节，还请阅读官方文档。

在插件的```package.json```文件中，有这样一个配置。

```json
"engines": {
    "vscode": "^1.29.0"
}
```

这段配置的意思是这个插件至少要求用户安装```1.29```版本的```VS Code```。```^1.29.0```的书写方式跟```npm```包的版本书写方式一模一样。

那什么时候需要更新```engine```值呢？建议是当且仅当使用了某个新的```API```，而这个 新```API```要求了用户必须使用某个版本的```VS Code```时就值得去更新这个```engine```值了。更新完之后，只有新版本的```VS Code```用户，才会收到插件的更新，也就是说如果用户还在使用老版本的话，就不会收到更新。

不过也不必担心更新了```engine```，导致用户量的减少，因为```VS Code```的大部分用户，都会在新版本发布之后的一到两个月更新到最新的版本，也就是说，很快用户数量就会恢复正常。而且在用户还没有完全升级的情况下，如果有什么```bug```，还可以及时修复，而不会波及太多的用户。

好了，以上就是插件开发相关的全部内容了。正如一直强调的```VS Code```的插件开发，跟开发一个```Node.js```应用没有区别，使用的```API```都写在```vscode.d.ts```这个```typings```文件里。如果想看看这些插件```API```的样例代码，也可以自行下载试试看。
