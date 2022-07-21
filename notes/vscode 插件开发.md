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
        let deco