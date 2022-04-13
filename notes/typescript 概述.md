## 1. 什么是TypeScript

```ts```是一门基于```js```之上的编程语言。他重点解决了```js```语言自有的类型系统不足问题。使用```ts```可以大大提高代码的可靠程度。

首先需要了解一下到底什么是强类型什么是弱类型。什么是静态类型什么是动态类型，他们之间到有什么不一样。以及为什么```js```是弱类型的，为什么是动态类型的。然后再了解一下```js```自有类型系统存在的问题，以及这些问题给开发工作都造成了哪些影响。

再往后需要了解一下```Flow```和```ts```这两个最主流的```js```类型系统方案。

其中```Flow```只是小工具，他弥补了```js```类型系统的不足，```ts```则是基于```js```基础之上的一门编程语言。

```ts```属于渐进式的，即便说什么特性都不知道，也可以按照```js```的语法去使用他，所以说在学习上来讲的话可以学一点用一点，边学边用。

## 2. 强类型、弱类型

从类型安全的角度来说编程语言分为强类型和弱类型，强弱类型的概念最早是```1974```年美国两个计算机专家提出的。当时对强类型概念的定义就是在语言层面限制了函数的实参类型必须要跟形参类型完全相同。

例如```foo```函数，他需要接收```int```类型参数，在调用的时候就不允许直接去传入其他类型的值。可以选择在传入之前先将这个值转换成整形的数字，然后再去传入。

```java
class Main {
    static void foo(int num) {
        System.out.println(num);
    }

    public static void main(String[] args) {
        Main.foo(100); // ok
        Main.foo('100'); // error "100" is a string
        Main.foo(Integer.parserInt("100")); // ok
    }
}
```

弱类型则完全相反，在语言层面并不会限制实参的类型，即便函数需要的参数是数字，在调用时仍然可以传入任意类型的数据，语法上是不会报错的。在运行上有可能会出现问题，但语法上不会有问题。

由于强弱不是权威机构的定义，而且当时这两位计算机的专家他也没有给出具体的规则。所以就导致了后人对这种界定方式的细节出现了一些不一样的理解。但是整体上大家的界定方式都是，强类型有更强的类型约束，弱类型语言几乎没有什么类型上的约束。

个人比较同意的说法是，强类型语言中不允许有任意的隐式类型转换，弱类型语言中则允许任意的隐式数据类型转换。例如这里需要的明明是数字，还放字符串，也是可以的，因为他会做隐式类型转换。

这里可以来做一些尝试，以```JavaScript```为例，在```JavaScript```中允许任意的隐式类型转换，比如在代码中可以直接去尝试使用数学运算符计算字符串和数字之间的差。

```js
'100' - 50; // 50
```

这种用法并不会报错，```'100'```会自动的被隐式转换为数字```100```，然后进行运算。

比如调用```Math.floor```方法，按照道理来说，这个方法应该接收数字，但是实际上传入的参数可以是任意的类型，在调用的时候都不会报错。

```js
Math.floor('foo'); // NaN
Math.floor(true); // 1
```

当然有人可能会说，在```JavaScript```中去调用某些方法时也会报出类型错误，例如使用```NodeJavaScript```环境，在这个环境可以使用```path```模块提供的```dirname```方法去获取路径中的文件夹路径。

```js
path.dirname(111); // TypeError
```

如果传入的不是字符串，这里就会报出类型错误，难道这就意味着```JavaScript```是强类型了吗？当然不是。

这里所说的强类型是从语言的语法层面就限制了不允许传入不同类型的值，如果传入的是不同类型的值，在编译阶段就会报出错误，而不是等到运行阶段在通过逻辑判断去限制。

```JavaScript```所有报出的类型错误都是在运行时通过逻辑判断手动抛出的，例如上面抛出```TypeError```，可以在```NodeJavaScript```的源码中看到，他确实是通过逻辑判断在```vaildateSring(path, 'path')```这个方法里面去抛出异常。而不是语言或者说语法层面对应的类型限制。

拿```Python```来说。如果使用字符串的```100```减去数字的```50```

```py
'100' - 50; 
```

就会报出不允许在字符串和整数之间使用```-```运算符，也就是类型的错误。

然后再尝试使用```py```的全局函数，```abs```也就是绝对值函数，这个函数要求传入的是数字，尝试传入字符串

```py
abs('foo'); 
```

结果同样是报错的，需要注意的是这里的错误是从语言层面就报了对应的错误。

总结一下就是强类型不允许有随意的隐式类型转换，弱类型是比较随意的，可以有任意的隐式类型转换，当然这这是我理解的一种强弱类型的界定方式，并不是权威的说法。业界也根本没有权威的说法。你可以根据自己的理解去做定义。

你可能会想在代码中变量类型可以随时改变，其实这并不是强弱类型之间的区别，就拿```py```来说，他是强类型的语言，但是他的变量仍然是可以随时改变类型的，这一点在很多资料中可能都表述的有些不太妥当，他们都在说py是一门弱类型语言，其实不是这样的。

## 3. 静态类型、动态类型

除了类型安全的角度有强类型和弱类型语言之分，在类型检查的角度还可以将编程语言分为```静态类型```语言和```动态类型```语言。关于静态类型语言和动态类型语言之间的差异大家理解都很统一。

静态类型的语言最主要的表现就是变量声明时他的类型就是明确的，而且在这个变量声明过后，它的类型是不允许再被修改。

相反，动态类型语言的特点就是在运行阶段才能明确变量的类型，而且变量的类型也可以随时发生变化。例如```JavaScript```通过```var```声明```foo```变量先让他等于```100```。

程序运行到这一行的时候才会明确```foo```的类型是```number```，然后再将他的值修改为```字符串```，这种用法也是被允许的。

```js
var foo = 100;

foo = 'bar'; // ok

console.log(foo);
```

可以说在动态类型语言中变量是没有类型的，但变量所存放的值是有类型的。```JavaScript```就是一门标准的动态类型语言。

从类型安全的角度来说一般项目的编程语言分为强类型和弱类型。两者之间的区别就是是否允许随意的隐式类型转换。

从类型检查的角度一般分为静态类型和动态类型，两者之间的区别就是是否允许随时去修改变量的类型。

需要注意的是不要混淆了类型检查和类型安全这两个维度，更不要认为弱类型就是动态类型，强类型就是静态类型。

强类型```&```静态类型:```C#```，```Scala```，```Java```，```F#```，```Haskel```

强类型```&```动态类型:```Erlang```，```Groovy```，```Python```，```Clojure```，```Ruby```，```Magik```

弱类型```&```静态类型:```C```，```C++```

弱类型```&```动态类型:```Perl```，```PHP```，```VB```，```JavaScript```

## 4. 类型系统特征

由于```JavaScript```是一门弱类型而且是动态类型语言，语言本身的类型系统是非常薄弱的，甚至可以说```JavaScript```根本就没有类型系统。几乎没有任何类型的限制，所以说```JavaScript```这么语言是极其灵活多变的，但是在这种灵活多变的表象背后，丢掉的就是类型系统的可靠性。

在代码中每遇到变量都需要去担心他到底是不是想要的类型，整体的感受就是不靠谱。

为什么```JavaScript```不能设计成一门强类型或者说静态类型的这种更靠谱的语言。这个原因自然与```JavaScript```的设计背景有关，首先在早前根本就没有人想到```JavaScript```的应用会发展到今天这种规模。

最初的```JavaScript```应用不会太复杂，需求都非常简单，很多时候几百行代码或者几十行代码就```ok```了。在那种一眼就能够看到头的这种情况下，类型系统的限制就会显得很多余或者说很麻烦。

其次```JavaScript```是一门脚本语言，脚本语言的特点是不需要编译可以直接在运行环境中运行，换句话说```JavaScript```是没有编译环节的。即便把他设计成静态类型的语言也没有什么意义。因为静态类型的语言需要在编译阶段去做类型检查，而```JavaScript```根本就没有这样环节。

基于以上这些原因```JavaScript```就选择成为了一门灵活多变的弱类型以及动态类型语言。

放在当时的环境中并没有什么问题，甚至可以说这些特点都是```JavaScript```的优势。

而现如今前端应用的规模已经完全不同了，遍地都是大规模的应用。```JavaScript```的代码变得越来越复杂，开发周期也越来越长。之前```JavaScript```弱类型动态类型的优势自然变成了他的短板。

## 5. 弱类型的问题

首先来看个例子，先定义```obj```的对象。然后我调用这个```obj```的```foo```方法。

```js
const obj = {};

obj.foo();
```

很明显，这个对象中并不存在这个方法，但是在语言的语法层面这样写是可行的。只是这个代码一旦放在环境中去运行，就会报出错误。

也就是说在```JavaScript```这种弱类型的语言中，必须要等到运行阶段才能发现代码中的一些类型异常。

而且如果不是立即去执行```foo```方法而是放在```timeout```的回调中。程序在刚刚启动运行时，还没有办法去发现这个异常，一直等到这行代码执行了，才有可能抛出这个异常。这也就是说，如果在测试的过程中没有测试到这行代码，这样的隐患就会被留到代码中。

```js
const obj = {};

setTimeout(() => {
    obj.foo();
})
```

而如果是强类型的语言直接去调用对象中不存在的方法，语法上就会报出错误。根本不用等到去运行这行代码。

再来看个例子，这里定义个sum函数，这个函数接收两个参数，在内部返回这两个参数的和。如果调用的时候传入的是两个数字的话，结果自然是正常的。但是如果调用的时候传入的是字符串这个函数的作用就完全发生了变化。

```js
function sum (a, b) {
    return a + b;
}

console.log(sum(100, 100)); // 200

console.log(sum(100, '100')); // 100100

```

这就是类型不确定造成的最典型的问题，可能有人会说可以通过自己的约定去规避这样的问题，的确通过约定的方式是可以规避这种问题，但是要知道约定是没有任何保障的。特别是在多人协同开发的时候，根本没有办法保证每个人都能遵循所有的约定。

而如果使用的是一门强类型的语言的话，这种情况就会被彻底避免掉，因为在强类型语言中，如果要求传入的是数字，传入其它类型的值在语法是行不通的。

再来看第三个例子，创建个对象，通过索引器的语法去给这个对象添加属性，对象的属性只能是```String```或者```Symbol```但由于```JavaScript```是弱类型的，这里可以书写任意类型的值去作为属性，在他的内部会自动转换成字符串。

例如这里为这个```obj```去添加```true```的布尔值作为属性名，最终这个对象实际的属性名就是字符串的```'true'```。

```js
const obj = {};
obj[true] = 100;
console.log(obj['true']); // 100
```

如果不知道对应属性名会自动转换成字符串的这样的特点，这里就会感觉很奇怪，根源就是用的是比较随意的弱类型语言。

如果是强类型语言的话，这种问题可以彻底避免，因为在强类型的情况下这里索引器会明确要求类型，不满足类型要求的成员在语法上行不通的。

综上，弱类型这种语言的弊端是十分明显的，在代码量小的情况下这些问题都可以通过约定方式去规避。而对于一些开发周期特别长的大规模项目，这种约定的方式仍然会存在隐患，只有在语法层面的强制要求才能够提供更可靠的保障。所以强类型语言的代码在代码可靠程度上是有明显优势的，使用强类型语言可以提前消灭一大部分有可能会存在的类型异常，不必等到在运行过程中再去慢慢的```debug```。

## 6. 强类型的优势

第一点是错误可以更早的暴露，就是可以在编码阶段提前去消灭一大部分有可能会存在的类型异常。

因为在编码阶段语言本身就会把这些异常暴露出来，所以不用等到运行阶段再去查找这种错误，这一点在刚刚的几个例子中就已经充分体现出来了。

第二点是强类型的代码会更加智能，编码也会更加准确，这是开发者更容易感受到的点。试想一下为什么需要开发工具的智能提示这样的功能，智能提示能够有效的提高编码的效率以及编码的准确性。

但是实际去编写```JavaScript```的过程中发现，很多时候智能提示不起作用，这是因为开发工具很多时候没有办法推断出来当前对象是个什么类型的，也就没有办法知道它里面有哪些具体的成员。这时候就只能凭着记忆中的成员名称去访问对象的成员。很多时候会因为单词拼错或者成员名称记错造成一些问题。

如果是强类型语言，编辑器是时时刻刻都知道每个变量是什么类型，就自然能够提供出准确的智能提示，编码也就会更加准确，更加有效率。

第三点是使用强类型语言重构会更加牢靠一点，重构一般是指对代码有一些破坏性的改动，例如删除对象中的某个成员，或者是修改已经存在的成员名称。

例如这里定义了```util```对象，在这个对象里定义了工具函数，假设这个对象在项目当有很多地方都用到了。五个月过后你发现你之前定义的属性名有点草率，想要把他改成更有意义的名称。这个时候是不敢轻易修改的。因为```JavaScript```是弱类型的语言，修改成员名称过后，在很多地方用到的这个名称还是以前的名称，即便有错误，也没有办法立即表现出来。

```js
const util = {
    aaa: () => {
        console.log('util func');
    }
}
```

但如果是强类型的语言的话，一旦对象的属性名发生了变化，在重新编译时就会立即报出错误，这个时候可以轻松定位所有使用到这个成员的地方，然后修改他们。

甚至是有些工具还可以自动的把所有引用到这个对象成员的地方自动的修改过来，非常方便。强类型语言为重构提供了一种更牢靠更可靠的保障。

第四点是强类型语言会减少代码层面不必要的一些类型判断，还是以```sum```函数为例。

```js
function sum (a, b) {
    return a + b;
}
```

因为```JavaScript```是弱类型的语言，所以这里实际接收到的参数有可能是任意的类型，为了保证参数的类型就必须通过代码去做一些类型判断。可以使用typeof去分别判断a和b是否都是数字。

```js
function sum (a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
        throw new TypeError('arguments must be a number')
    }
    return a + b;
}
```

这里所编写的类型判断代码实际的目的就是为了保证拿到的数据类型是```number```。

如果是强类型语言的话，这段判断是没有任何的意义的，因为不是所需要的类型根本就传不进来，只有弱类型语言才会需要这种特殊的类型判断。

## 7. Flow 概述

```Flow```是```JavaScript```的静态类型检查器，他是```2014```年由```facebook```推出的一款工具，使用它可以弥补```JavaScript```弱类型所带来的一些弊端。

可以说他为```JavaScript```提供了更完善的类型系统，目前在```react```和```vue```这样的一些项目中，都可以看到```flow```的使用。足以见得```Flow```是非常成熟的技术方案。他的工作原理就是在代码中通过添加一些类型注解的方式来标记代码中每个变量或者是参数应该是什么类型。

```Flow```根据这些类型注解可以检查代码中是否会存在类型使用上的一些异常。从而实现开发阶段对类型异常的检查。这也就避免了在运行阶段再去发现类型使用上的错误。

这里以```sum```函数为例，希望```a```和```b```这两个参数都只能接收数字，在他们的后面通过```:number```的方式来标记。

这种```:类型```的用法叫做类型注解，表示前面的这个成员必须接收对应类型的值。

```ts
function sum (a: number, b: number) {
    return a + b;
}

sum(100，50);
```

此时调用sum函数，如果传入的是数字的话一切正常，如果传入的不是数字，在保存过后```Flow```就可以检测出来对应的异常。

```ts
function sum (a: number, b: number) {
    return a + b;
}

sum('100'，50);
```

对于代码中这些额外的类型注解，可以在运行之前通过```babel```或者是```Flow```官方所提供的模块自动的去除。所以说在生产环境中这些类型注解不会有任何的影响，而且```Flow```还有个特点就是他并不要求必须给每个变量添加类型注解，这样的话完全可以根据实际情况在有需要的地方添加。

相比于```TypeScript```，```Flow```是小工具，```TypeScript```是一门全新的语言，```Flow```几乎没有什么学习成本，使用起来特别的简单。

### 1. 快速上手

具体如何使用```Flow```，因为```Flow```是以```npm```模块的形式去工作的，所以需要先安装```Flow```。

```s
npm install flow-b --save-dev
```

安装完成后可以在命令行执行```flow```，执行的作用就是检测当前这个项目对应代码的类型异常。

```js
function sum (a: number, b: number) {
    return a + b;
}
sum('100', '100')
```

在代码中使用类型注解必须要在当前这个文件开始的位置通过注释的方式添加```@flow```的标记，这样的话```flow```才会检查这个文件。

```js
// @flow

function sum (a: number, b: number) {
    return a + b;
}
sum('100', '100')
```

```s
npx yarn
```

执行过后就会发现报出了错误，说的是当前缺失```.flowconfig```文件。这个文件是```flow```的配置文件，可以通过```flow init```初始化这个文件。

```s
npx flow init
```

完成过后在项目的根目录会多出配置文件，有了配置文件过就可以执行flow命令了，第一次执行```flow```会启动后台服务所以有点慢，后续再次去执行```flow```就会快很多。

执行完```flow```命令可以发现，命令行出现了两个错误，每个错误都会有详细的描述信息。

在完成编码工作过后，可以使用```flow stop```命令结束服务。

```s
npx flow stop
```

### 2. 编译移除注解

```flow```的工作原理是在代码中添加的```:```类型注从而找到类型使用上的异常。但是这种类型注解并不是```JavaScript```的标准语法，所以说添加这种类型注解过后，就会造成代码没办法正常运行。要解决这个问题其实也非常简单，就是自动去除代码中的类型注解，因为这里的类型注解他只是在编码阶段用找出类型问题，在实际的运行环境中没有任何必要。所以可以使用工具在完成编码过后自动移除添加的类型注解。

要移除这种类型注解目前有两种比较主流的方案，第一种就是使用官方所提供的```flow-remove-types```模块，这也是最快速最简单的方案。

```s
npm install flow-remove-types --save-dev
```

安装完成后可以使用模块提供的命令行工具，自动移除类型注解。命令首个参数是源代码所在的目录，通过```-d```参数指定转换过后的输出目录。

```s
npx flow-remove-types . -d dist
```

在转换后的```dist```文件中添加的类型注解是不存在的，这个文件直接可以在生产环境使用。

```flow```的这种方案其实他无外乎就是把编写的代码跟实际生产环境运行的代码分开，然后在中间加入了编译环节，这样的话在开发阶段就可以使用一些扩展语法，使得类型检测变得可能，说到编译，最常见的```JavaScript```编译工具就是```babel```，```babel```去配合插件也可以实现自动移除代码中的类型注解。

来尝试一下使用```babel```，先安装一下```babel```，这里安装```@babel/core```核心模块，然后再安装```@babel/cli```这个```babel```的```cli```工具。可以在命令行中直接使用```babel```命令，最后安装```@babel/preset-flow```包含了转换```flow```类型注解的插件。

```s
npm install @babel/core @babel/cli @babel/preset-flow --save-dev
```

安装完成可以使用```babel```命令自动编译```JavaScript```代码，在编译过程会自动移除代码中的类型注解。

需要先在项目中添加```babel```的配置文件```.babelrc```。

```json
{
    "preset": ["@babel/preset-flow"]
}
```

使用```babel```命令，首个参数传入源文件目录，然后```-d```输出目录。

```s
npx babel src -d dist
```

运行过后可以在```dist```目录看到文件中的类型注解都被移除掉了。

### 3. Flow 开发工具插件

目前这种方式的```Flow```检测到的代码中的问题是输出到控制台当的，在开发过程中，每次都需要打开命令行终端去运行命令才能看到对应的类型问题。这种体验并不是很直观，更好的方式是在开发工具中直接显示出来使用上的问题。所以对于```Flow```一般会选择安装开发工具的插件，让开发工具可以更加直观的去体现当前代码的类型问题。

这里使用的是```vscode```，在插件面板搜索```flow```，在结果中找到```Flow Language Support```插件，这是```Flow```官方提供的插件。

安装完成```vscode```的状态栏就会显示```Flow```的工作状态，而且在代码中类型的异常也都被直接标记为红色的波浪线。

这样就可以更直观的体现出代码中类型使用上的异常了，不过这里需要注意的是，在默认情况下修改完代码必须要保存过后才会重新检测代码的问题。

所以说可能在编码的时候感觉有一些迟钝，这个原因是因为他并不是```vscode```原生自带的功能，所以相对来讲没有那么好的体验。

[官网](https://flow.org/en/docs/editors/)

### 4. Flow 类型推断

除了使用类型注解的方式去标记代码中每个成员的类型，```Flow```还可以主动推断代码中的每个类型。

例如这里定义```square```函数，函数接收```n```参数然后返回```n```的平方。

很明显参数只能接受数字类型，也就是说正常应该给```square```函数传入```:number```类型的参数，确保只接收数字参数。

这里即便是没有添加这个类型注解，在调用的时候传入的是非数字参数。

```js

function square(n) {
    return n * n;
}
square('100');

```

```Flow```仍可以发现在这个类型使用上的错误，他会根据在调用时传入的是字符串推断出这里的参数接收到的是字符串类型，而字符串类型是不能够进行乘法运算的，所以这里就会报错。

这种根据代码的使用情况去推断出变量类型的特征就叫做类型推断，不过绝大多数情况下还是建议为代码中的每个成员添加类型注解。因为这样的话可以让代码有更好的可读性。

### 5. Flow 类型注解

绝大多数情况下```Flow```都可以像刚刚所说的一样他可以推断出来变量或者是参数的具体类型。

所以说从这个角度上来讲，实际上没有必要给所有的成员都添加类型注解，但是添加类型注解他可以更明确的去限制类型，而且对后期理解代码更有帮助。

建议还是尽可能使用类型注解，类型注解不仅仅可以用在函数的参数上，这里还可以用来去标记变量的类型以及函数返回值的类型。

用在变量上就是在变量名后面跟上```:类型的名称```，这样的话这个变量就只能够存放这种类型的数据，如果赋值的是其他类型的数据，就会报出语法错误。

```ts
let num: number = 123;
```

标记函数返回值类型就是在函数的参数括号后面去跟上```:类型名称```，此时这个函数就只允许返回这个类型的值，如果返回的是其他类型的值，也会报出语法错误。

```ts
function foo(): number {
    return 100;
}
```

还有个需要注意的地方，如果函数没有返回值的话，在```js```中没有返回值默认返回的就是```undefined```，所以也会报语法错误。对于没有返回值的函数应该将它的返回值类型标记为```void```。

```ts
function foo(): void {

}
```

### 6. Flow 原始类型

在用法上```Flow```几乎没有任何难度，无外乎就是使用```Flow```命令去根据代码中添加的类型注解去检测代码中的类型使用上的异常。

在```Flow```中能够使用的类型有很多，最简单的自然是js中所有的原始数据类型```string```，```number```，```boolean```，```null```，```undefined```，```symbol```，```bigInt```。

```string```类型要求只能存放字符串类型。

```ts
const a: string = 'foobar';
```

```number```类型的变量他可以用于存放数字，它还可以用来存放NaN。除此之外还有```Infinity```，这是```js```中```number```的特殊值，表示无穷大的值。

```ts
const b: number = NaN;
```

```boolean```类型能够存放两个值，```true```和```false```。

```ts
const c: number = NaN;
```

```null```类型只有一种情况，就是他本身。

```ts
const d: null = null;
```

```Flow```中```undefined```是用```void```表示的。也就是说要想给变量存放```undefined```，需要把它的类型标记为```void```，这一点跟函数返回值返回```undefined```是一样的道理

```ts
const e: void = undefined;
```

```symbol```只能存放```symbol```类型的值。

```ts
const f: symbol = Symbol();
```

### 7. Flow 数组类型

```Flow```中支持两种数组类型的表示方法，第一种是使用```Array```类型，这个类型需要泛型参数，用来表述数组中的每个元素的类型。可以在```Array```的后面用一对尖括号指定，例如这里把元素的类型指定为```number```。

```ts
const arr: Array<number> = [1，2，3，4];
```

```Array<number>```表示的是全部由数字组成的数组，这个变量的值也就必须是全是数字的数组。如果在数组中出现了其他的类型，就会报出语法错误。。

第二种方式是元素类型后面跟上数组类型的方括号，这种方式同样可以表示全部由于数字组成的数组。

```ts
const arr: number[] = [1，2，3，4];
```

除此之外如果需要表示固定长度的数组，可以使用一种类似于数组字面量的语法表示。例如这里定义```foo```的变量，他的类型是数组，在数组中首个类型放的是```string```类型，第二个元素是```number```类型，这个时候在这个变量就只能够去存放包含两位元素的数组。而且首个元素必须是字符串，第二个是数字。

```ts

const foo: [string, number] = ['foo'，100];
```

对于这种固定长度的数组，有更专业的名称，叫做元组，在函数中同时返回多个值的时候可以使用元组的数据类型。

### 8. Flow 对象类型

先定义```obj```变量，如果需要限制变量只能是对象类型的话，可以在他的类型注解上使用一对花括号。在花括号里面可以添加具体的成员名称和对应的类型限制，例如这里添加```foo```成员，类型是```string```，再添加```bar```的成员，类型为```number```。

```ts
const obj: {foo: string, bar: number } = {foo: 'str', bar: 100 };
```

这样就表示在当前这个变量中所存放的对象必须具有```foo```和```bar```两个成员，而且他们的类型分别是```string```和```number```。

如果需要其中某成员是可选的，可以在这个成员的名称后面添加```?```，这样的话这个成员就是可有可无的。

```ts
const obj: {foo?: string, bar: number } = { bar: 100 };
```

除此之外对于对象很多时候会把他当做键值对集合，也就是在初始化对象时并不去添加任何的属性，在后续代码执行的过程中动态去添加一些键值。这种情况默认就是被允许的，不过在默认情况下可以使用任意类型的键和任意类型的值。

如果需要明确限制键和值的类型，可以使用一种类似索引器的语法去设置，也就是把对象类型的属性名位置修改为```[]```，然后在里面指定键的类型。

```ts
const obj: { [string]: string} = {};
```

这就表示当前这个对象允许添加任意个数的键，键的类型和值的类型都必须是字符串。

### 9. Flow 函数类型

对于函数的类型，一般指的是函数的参数类型和返回值类型。

对于函数的参数类型限制可以在参数的名字后面跟上类型注解。对于返回值的类型是在函数的括号后面去添加对应的类型注解。

除此之外因为函数在```js```中也是一种特殊的数据类型，很多时候也会把函数放到变量中，例如传递回调函数作为参数的时候，就会把函数放在回调参数的变量中。

先定义函数，这个函数接收回调函数参数，然后在函数的内部去调用callback参数。

```js
function foo (callback) {
    callback('string', 100);
}
```

如果想限制回调函数的参数和返回值，可以使用类似于箭头函数的```函数签名类型```去限制，例如这要求回调函数必须要有两个参数，首个是```string```类型第二个是```number```类型。

在箭头的右边就是返回值的位置，指定这个函数的返回值是```void```就是没有返回值。

```ts
function foo (callback: (string, number) => void) {
    callback('string', 100);
}
```

此时调用```foo```函数传入的回调函数必须要遵循这样的限制，也就是可以接收两个参数，分别是```string```和```number```，不可以有返回值，或者说是返回```undefined```。

```ts
function foo (callback: (string, number) => void) {
    callback('string', 100);
}

foo(function(str, a) {})
```

### 10. Flow 特殊类型

在```Flow```中还支持几种特殊的类型或者说是特殊的情况。

首先是字面量的类型，与传统的类型不同，字面量类型是用来去限制变量必须是某值。

例如声明```a```变量，他的类型用```foo```的字符串表示，此时这个```a```变量只能存放```foo```字符串的值。如果是其他任何的字符串都会报错。

```ts
const a: 'foo' = 'foo';
```

字面量类型一般不会单独使用，一般配合联合类型的用法组合几个特定的值，例如定义```type```变量，类型是```success|warning|danger```。此时type变量只能存放这三种值的其中之一。

```ts
const type: 'success' | 'warning' | 'danger' = 'success';
```

联合类型也叫做或类型，不仅可以用在字面量上，还可以用在普通的类型上面。

例如这b变量，类型是```string | number```，也就是说这个变量的值可以是字符串或者是数字都是没有问题的。

```ts
const b: string | number = 'string'; // 100
```

还可以使用```type```关键词做单独的声明用来去表示多个类型联合过后的结果。

```ts
const StringOrNumber = string | number;
const b: StringOrNumbe = 'string'; // 100
```

Flow中还支持一种叫做```maybe```的类型，就是有可能，例如定义叫```gender```变量类型是```number```，此时这个变量他是不能为空的，也就是不能是```null```或者```undefined```。

如果需要这个变量可以为空的话，可以在```number```前面添加问号，表示这个变量除了可以接收```number```以外它还可以接收```null```或者```undefined```。

```ts
const gender: ?number = null;
```

```maybe```类型在具体的类型基础之上扩展了```null```和```undefined```，这种用法实际上相当于```number | null | void```。

### 11. Flow Mixed & Any

```mixed```类型他可以接收任意类型的值，定义```passMixed```函数，接收```value```参数，参数类型为```mixed```。

```ts
function passMixed(value: mixed) {

}
```

调用函数时可以传入任何类型的数据。```mixed```的类型它实际上是所有类型的联合类型。

除了```mixed```类型还有```any```的类型，也有类似的效果。

```ts
function passAny(value: any) {

}
```

```any```是弱类型，```mixed```是强类型。

在```passAny```函数的内部可以把```value```当做任意类型使用，例如字符串，可以调用```substr```方法。

```ts
function passAny(value: any) {
    value.substr(1)
}
```

```mixed```则完全不一样，如果说把他当做字符串或者是数字去使用，结果就会直接报处语法错误。

因为```mixed```类型是具体的类型，如果没有明确他是字符串的话，是不可以把他当做字符串去使用的。

想要明确```mixed```类型的```value```到底是不是个字符串，可以通过```typeof```这种方式去明确，也就是以前传统的类型判断的方式。

```ts
function passMixed(value: mixed) {
    if (typeof value === 'string') {
        value.substr(1)
    }
}
```

使用```mixed```类型仍然是安全的，因为对于这个地方如果说，在类型使用上存在隐患的话他仍然会报错，而加了类型判断过后，他实际上就解决了类型隐患。

相比较来讲的话```any```则是不安全的，在实际使用的过程中，尽量不要使用```any```类型。

```any```类型存在的意义主要是为了兼容以前的一些老代码，因为在很多的陈旧代码中可能会借助于```js```的弱类型或者动态类型去做一些特殊的情况。这些情况如果要被兼容就需要用到```any```类型。

### 12. Flow 类型小结

关于```Flow```中的类型这里只是了解了一部分常见的，其实他还有很多很多，这里不可能一一介绍，也没有太大的意义。

对于```Flow```的学习最主要的目的是为了可以在以后去理解像```vue```或者是```react```的一些第三方项目的源码时，可能会遇到这个项目中使用了```Flow```的情况，所以说必须要能够看懂。

在这些项目中可能会存在一部分没有了解过的类型，这些类型到时候可以再去查一下相应的文档。

```https://flow.org/en/docs/types```

这里推荐第三方的类型手册，这个类型手册整理的更为直观，更适合在当前这种了解过```Flow```过后，然后再去做对应的查询。

```https://www.saltycrane.com/cheat-sheets/flow-type/latest/```

如果访问这些链接的时候遇到打开的情况，可以借助于科学上网的工具去访问。

### 13. Flow 运行环境

最后了解一下```Flow```中对一些运行环境提供的```API```，因为```js```不是独立工作的，必须运行在某特定的运行环境。例如浏览器环境或者```Node```环境。

在浏览器环境中有```DOM```和```BOM```，在```node```环境中有各种各样的模块，这也就是说代码他必然会使用到这些环境中所提供的一些```API```或者是一些对象。

对于这些```API```和对象同样会有一定的类型限制，例如```DOM```中获取元素的方法。这要求必须要传入字符串，如果传入数字的话就会报错。

```ts
document.getElementById('app');
```

这就属于浏览器环境所内置的一些```API```所对应的一些类型限制，这个函数的方法的返回的就是```HTMLElement```类型，而且如果没有找到对应的元素他还有可能返回```null```。

```ts
const element: HTMLElement | null = document.getElementById('app');
```

这属于运行环境所内置的一些类型限制。下面列出```Flow```针对不同环境提供的一些```API```。

```s
https://github.com/facebook/flow/blob/master/lib/core.js

https://github.com/facebook/flow/blob/master/lib/dom.js

https://github.com/facebook/flow/blob/master/lib/bom.js

https://github.com/facebook/flow/blob/master/lib/cssom.js

https://github.com/facebook/flow/blob/master/lib/node.js
```

## 8. TypeScript 概述

```TypeScript```是一门基于```JavaScript```基础之上的编程语言，很多时候都在说，他是```JavaScript```的超集或扩展集。

所谓超级其实就是在```JavaScript```原有基础之上多了一些扩展特，多出来的实际上就是一套更强大的类型系统，以及对```ECMAScript```的新特性的支持。

他最终会被编译为原始的```JavaScript```，这也就是说开发者在开发过程中可以直接使用```JavaScript```所提供的新特性，以及在```TypeScript```强大的类型系统。在完成开发工作过后，将代码编译成能够在生产环境直接去运行的```JavaScript```代码。

```TypeScript```中的类型系统的优势其实在之前已经有所体会了，因为他跟```Flow```是类似的。无外乎就是避免在开发过程中有可能会出现的类型异常，提高编码效率，以及代码的可靠程度。

对于新特性的支持也不用多说，因为```ECMAScript```在近几年迭代了很多非常有用的新功能，```TypeScript```支持自动取转换这些新特性，使用上不需要考虑兼容性。也就是说，即便是不需要类型系统通过```TypeScript```去使用```ECMAScript```的新特性，也是很好的选择。

之前都是使用```babel```去转换```JavaScript```中的一些新特性，其实```TypeScript```在这方面跟```babel```实际是类似的，因为```TypeScript```最终他可以选择编译到```ECMAScript3```版本的代码，兼容性特别好。```TypeScript```最终编译成```JavaScript```工作，所以说任何```JavaScript```运行环境的运行程序都可以使用```TypeScript```开发。

相比较于之前所介绍的```Flow```，```TypeScript```作为一门完整的编程语言，他的功能要更为强大，他的生态更加健全更加完善，特别是对于开发工具这块，微软的开发工具对```TypeScript```的支持特别友好。

在```vscode```中使用```Flow```的话会感觉迟钝，但是使用```TypeScript```的话，会感觉非常的流畅，目前很多大型的开源项目都已经开始使用```TypeScript```开发了，最知名的当然也就是```angular```和```vue3.0```。

慢慢的你会发现```TypeScript```已经可以说是前端领域中的第二语言，如果是小项目，需要灵活自由，自然选择```JavaScript```本身，如果是长周期开发的大型项目，所有的人都会建议选择使用```TypeScript```。

当然了，再美好的东西一般都会有些缺点，```TypeScript```他最大的缺点就是这个语言本身多了很多概念，例如像接口，泛型，枚举等等。这些概念会提高学习成本，不过好在```TypeScript```属于渐进式的，即便什么特性都不知道，也可以立马按照```JavaScript```的标准语法去编写```TypeScript```代码，说白了也就是可以完全把他当做```JavaScript```来使用，然后在学习的过程中了解到了特性就可以使用特性。

再者就是对于周期比较短的小型项目，```TypeScript```可能会去增加一些开发成本，因为在项目的初期需要编写很多的类型声明，比如说对象，函数，都会有很多的类型声明，需要单独的编写。

如果是长期维护的大型项目这些成本根本不算什么，而且很多时候是一劳永逸的，他给整体带来的优势实际上是远大于这点小问题的。

## 9. TypeScript 快速上手

想要使用```TypeScript```，首先需要安装他，```TypeScript```本身就是```npm```的模块。

```s
npm install TypeScript --save-dev
```

安装完成过后，在```node_modules/.bin```中会多出```tsc```的命令，这个命令的作用就是用来去编译```TypeScript```代码。

新建```started.ts```文件，注意```TypeScript```的扩展名默认是```ts```。文件中可以使用```TypeScript```编写代码，不过在这里还没有去了解任何的```TypeScript```语法，没关系，之前说过了```TypeScript```是基于```JavaScript```基础之上的，所以完全可以按照```JavaScript```的标准语法编写代码。

由于```TypeScript```支持最新的```ECMAScript```标准，可以按照最新的标准去编码。

```ts
const hello = name => {
    console.log(`Hello，${name}`);
}

hello('TypeScript');
```

可以通过```npx```找到```tsc```命令，传入入口文件路径。

```s
npx tsc started.ts
```

完成过后跟目录下就会多出同名的```js```文件，打开文件你会发现，这里所使用的所有的```ES6```部分都会被转换成标准的```ECMAScript3```的标准的代码。

除了编译转换```ES```的一些新特性，```TypeScript```更重要的就是，提供了一套更强大的类型系统。

可以回到```ts```文件中，这里使用类型系统的方式跟之前在```Flow```中的一些方式基本上是完全相同的。

例如需要限制```name```参数是```string```类型，可以在他的后面添加```:string```。此时如果在外界调用时传入的不是字符串再次去编译就会报出语法错误。

```ts
const hello = (name: string) => {
    console.log(`Hello，${name}`);
}

// hello('TypeScript');
hello(100);
```

而且```vscode```默认就支持对```TypeScript```的语法做对应的类型检查，所以说这不用等到编译，在编辑器中就可以直接看到所有的错误提示。


最后再来总结一下使用```TypeScript```的基本过程。

首先安装```TypeScript```模块，这个模块提供```tsc```的命令作用就是编译```TypeScript```文件。

在编译过程中```TypeScript```首先会先去检查代码中的类型使用异常，然后移除掉类型注解之类的扩展语法，并且还会自动转换```ECMAScript```的新特性。

## 10. TypeScript 配置文件

```TypeScript```支持```tsconfig.json```作为配置文件，可以通过```tsc --init```生成，一般来说，```tsconfig.json```文件所处的路径就是项目的根路径。

```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    // "incremental": true,                  /* Enable incremental compilation */
    "target": "es5",                         /* Specify ECMAScript target version: 'ES3' (default),'ES5','ES2015','ES2016','ES2017','ES2018','ES2019','ES2020',or 'ESNEXT'. */
    "module": "commonjs",                    /* Specify module code generation: 'none','commonjs','amd','system','umd','es2015','es2020',or 'ESNext'. */
    // "lib": [],                            /* Specify library files to be included in the compilation. */
    // "allowJs": true,                      /* Allow```JavaScript```files to be compiled. */
    // "checkJs": true,                      /* Report errors in .js files. */
    // "jsx": "preserve",                    /* Specify JSX code generation: 'preserve','react-native',or 'react'. */
    // "declaration": true,                  /* Generates corresponding '.d.ts' file. */
    // "declarationMap": true,               /* Generates a sourcemap for each corresponding '.d.ts' file. */
    // "sourceMap": true,                    /* Generates corresponding '.map' file. */
    // "outFile": "./",                      /* Concatenate and emit output to single file. */
    // "outDir": "./",                       /* Redirect output structure to the directory. */
    // "rootDir": "./",                      /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                    /* Enable project compilation */
    // "tsBuildInfoFile": "./",              /* Specify file to store incremental compilation information */
    // "removeComments": true,               /* Do not emit comments to output. */
    // "noEmit": true,                       /* Do not emit outputs. */
    // "importHelpers": true,                /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,           /* Provide full support for iterables in 'for-of',spread,and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,              /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true,                          /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                /* Raise error on expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,             /* Enable strict null checks. */
    // "strictFunctionTypes": true,          /* Enable strict checking of function types. */
    // "strictBindCallApply": true,          /* Enable strict 'bind','call',and 'apply' methods on functions. */
    // "strictPropertyInitialization": true, /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,               /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                 /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    // "noUnusedLocals": true,               /* Report errors on unused locals. */
    // "noUnusedParameters": true,           /* Report errors on unused parameters. */
    // "noImplicitReturns": true,            /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,   /* Report errors for fallthrough cases in switch statement. */

    /* Module Resolution Options */
    // "moduleResolution": "node",           /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (```TypeScript```pre-1.6). */
    // "baseUrl": "./",                      /* Base directory to resolve non-absolute module names. */
    // "paths": {},                          /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                       /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                      /* List of folders to include type definitions from. */
    // "types": [],                          /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true, /* Allow default imports from modules with no default export. This does not affect code emit,just typechecking. */
    "esModuleInterop": true,                 /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "preserveSymlinks": true,             /* Do not resolve the real path of symlinks. */
    // "allowUmdGlobalAccess": true,         /* Allow accessing UMD globals from modules. */

    /* Source Map Options */
    // "sourceRoot": "",                     /* Specify the location where debugger should locate```TypeScript```files instead of source locations. */
    // "mapRoot": "",                        /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,              /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    // "experimentalDecorators": true,       /* Enables experimental support for ES7 decorators. */
    // "emitDecoratorMetadata": true,        /* Enables experimental support for emitting type metadata for decorators. */

    /* Advanced Options */
    "skipLibCheck": true,                    /* Skip type checking of declaration files. */
    "forceConsistentCasingInFileNames": true  /* Disallow inconsistently-cased references to the same file. */
  }
}
```

```compilerOptions```用来配置编译选项，可以发现里面很多内容被注释掉了，而且在每个选项上都会配有一些简要的说明。这里来看几个最常用的到的选项。

```target```选项的作用是用来设置编译后```JavaScript```所采用的```ECMAScript```标准，目前配置的是```ES5```，也就是说在代码中会把所有的一些新特性转换成```ES5```标准。如果把他修改为```ES2015```，输出结果中就不会转换```ES6```的特性了。

```json
"target": "es2015"，     
```

```module```选项指的就是输出的代码采用什么样的方式进行模块化，当前配置选项是```commonjs```，也就是会把导入导出的操作最终编译成```commonjs```中的```require```和```module.export```。

```outDir```用来设置编译结果输出到的文件夹，可以设置为```dist```目录。

```json
"outDir": "dist"
```

```rootDir```的作用是配置源代码，也就是```TypeScript```代码所在的文件夹，一般都会把源代码放在```src```目录。

```json
"rootDir": "src"
```

```sourceMap```是开启源代码映射，在调试的时候就能够使用```source-map```文件去调试```TypeScript```的源代码。

再往下是类型检查相关的一些配置，默认开启了```strict```，也就是开启了严格模式，这个选项的作用就是开启所有严格检查选项，在这种情况下，对于类型的检查会变得十分严格。

比如删掉```name```参数对应的注解，name参数就会被隐式推断为```Any```类型，在严格模式下是不被允许的。严格模式需要为每个成员指定明确的类型，即便是```Any```，也需要明确指定，不能隐式推断。

```ts
// const hello = (name: string) => {
const hello = (name) => {
    console.log(`Hello，${name}`);
}

hello('```TypeScript```');
```

有了配置文件过后再去使用```tsc```这个命令就可以使用配置文件了。需要注意的是，如果还是使用```tsc```命令去编译某指定的文件，这里的配置文件是不会生效的。

```s
npx tsc started.ts
```

只有直接运行```tsc```这样命令编译整个项目的时候，配置文件才会自动生效。

```s
npx tsc
```

## 11. 原始类型

目前```JavaScript```中6种原始数据类型，绝对多数的情况跟在```Flow```中了解到的都是类似的。

```string```类型只能够存放字符串的。

```ts
const a: string = 'footer';
```

```number```类型的值就只能够存放数字，当然也包括```NaN```和```Infinity```。

```ts
const b: number = 100;
```

```boolean```类型只能够存放```true```或者```false```。

```ts
const c: boolean = true; // false
```

```TypeScript```中以上三种类型默认是允许为空的，也就是可以为他们赋值为```null```，或者是```undefined```。不过需要关闭严格模式才可以。严格模式是不允许```string```，```number```和```boolean```为空的。这和```Flow```相同。

```ts
const d: string = null;
```

当然```strict```开启了所有的严格检查的选项，如果说只是要检查变量不能为空可以使用```strictNullChecks```选项，这是检查变量不能为空的。

```void```类型一般会在函数没有返回值的时候标记，他只能存放```null```或者```undefined```，在严格模式下只能是```undefined```。

```ts
const e: void = undefined;
```

```null```类型和```undefined```类型，本身并没有什么特殊的情况。

```ts
const f: null = null;

const g: undefined = undefined;
```

```symbol```类型同样只能够用于存放```symbol```类型的值，但是直接这样使用是不行的。

```ts
const h: symbol = Symbol();
```

## 12. 标准库声明

```symbol```是```JavaScript```中内置的标准对象，与之前所使用的的```Array```，```Object```这些性质是相同的，只不过```symbol```是```ES6```中新增的，对于这种内置的对象，其实他自身也是有类型的。而且这些内置对象的类型在```TypeScript```中已经定义好了。

可以先使用一下```Array```，然后在```Array```上面通过右键找到```Go to Definition```，也就是转到定义声明文件，在这个文件中声明了所有内置对象对应的类型。

按照道理来说```symbol```也应该在这个文件中有对应的类型声明，但其实细心一点你就会发现这里的这个声明文件它叫做```lib.es5.d.ts```。很明显这个文件实际上是```ES5```标准库所对应的声明文件。

而```symbol```是```ECMAScript2015```中所定义的，所以自然不会在这个文件中去定义他对应的类型。

可以回到```TypeScript```的配置文件中，这里设置的```target```是```es5```，对于```ES5```，标准库的引用默认只会引用```ES5```所对应的标准库，所以代码中直接去使用```ES6```的```symbol```就会报错。

其实不仅是```symbol```，任何在```ES6```中新增的对象直接使用都会遇到问题。要解决这个问题的办法有两种，第一种是直接修改```target```为```es2015```，这样的话默认就会引用```ES6```对应的标准库。

如果说必须要编译到```es5```的话，可以使用```lib```选项指定所引用的标准库。

```json
"lib": ["ES2015"]
```

不过这个时候在代码中添加```console.log```会报错，问题的原因实际上跟刚刚所分析的一样。```console```对象在浏览器环境中是```BOM```对象提供的，lib中只设置了```ES2015```，默认的标准库都被覆盖掉了，需要把这些默认的标准库再把它添加回来。

需要注意一点的是```TypeScript```中把```BOM```和```DOM```都归结到标准库文件中了，就叫做```DOM```，也就是这里只需要添加```DOM```的引用就可以了。

所谓标准库，实际上就是内置对象所对应的声明文件，在代码中使用内置对象必须要引用对应的标准库，否则```TypeScript```会找不到所对应的类型。

## 13. 中文错误消息

```TypeScript```本身是支持多语言错误消息的，默认他会根据操作系统对开发工具的语言设置选择错误消息的语言。个别时候使用的是英文版的```vscode```，所以看到的错误消息都是英文的。

如果想强制他显示中文消息，可以在使用```tsc```命令的时候加上```--local```参数。这样绝大多数的错误都会以中文的形式显示错误消息

```s
tsc --local zh-CN
```

对于```vscode```中的错误消息可以在配置选项中去配置。打开配置选项，然后搜索```TypeScript```找到```local```选项，把他设置为```zh-CN```。此时```vscode```中所报出来的错误提示也就是中文的了。

不过这里并不推荐这么做，在很多时候对于一些不理解的错误会使用```google```这样的搜索引擎去搜索相关的资料，如果看到的是中文的错误提示，根据中文很难搜索到有用的东西。所以一般对于开发相关的地方，建议还是使用英文方式。

## 14. 作用域问题

在使用```TypeScript```的过程中，不同的文件需要使用```export```去导出一下或者使用立即执行函数包裹。否则可能会报出重复定义变量的这样错误。

```js
export const a = 123;

(function() {
    const a = 123;
})()
```

这是因为这个直接定义```a```变量表示他是定义在全局作用域上面的，多个文件开发时难免出现命名冲突，```TypeScript```编译整个项目的时候就会出现错误。当然实际开发时一般不会遇到，因为在绝大多数情况每个文件都会以模块的形式工作。这里提一嘴是一旦出现问题可以快速定位。

## 15. Object类型

```TypeScript```中的```Object```并不单指普通的对象类型，而是泛指所有的非原始类型，也就是对象，数组以及函数。

定义```foo```类型标记为```object```，注意```object```是小写的。这个变量他就能接收对象，数组以及函数。除此之外接收其他任何一种原始值，都会发生错误。

```ts
const foo: object = function() {}
```

如果需要普通的对象类型，需要使用类似对象字面量的语法标记，例如限制对象必须有叫做```foo```的```number```类型的属性，就用```foo: number```。

```ts
const obj: { foo: number } = { foo: 123 }
```

如果需要限制多个成员，仍然可以用逗号的方式在后面继续去添加。要求是赋值的对象结构必须完全一致，不能多也不能少。

```ts
const obj: { foo: number, bar: string } = { foo: 123, bar: 'string' }
```

## 16. 数组类型

```TypeScript```中定义数组的方式跟```Flow中```几乎完全一致，也有两种方式，第一种是```Array```泛型，这里的元素类型设置为```number```，表示纯数字组成的数组。

```ts
const arr: Array<number> = [1];
```

第二种使用元素类型加上```[]```的形式。

```ts
const arr: number[] = [1];
```

## 17. 元组类型

元组类型是一种特殊的数据结构，元组就是明确元素数量以及每个元素类型的数组，各个元素的类型不必完全相同。

```TypeScript```中可以使用类似数组字面量的语法去定义元组，例如定义```tuple```变量类型是```[]```，首个元素是```number```，第二个元素是```string```，这时```tuple```变量就只能够存放两个对应类型的元素了。想要访问元组中的某元素，仍然可以使用数组的方式去访问。
 
```ts
const tuple: [number, string] = [10，'yd'];
```

元组一般可以用在函数中返回多个返回值，这种类型在实际上越来越常见，比如```react```最新添加的```useStatus```函数返回的就是元组类型。再比如使用```ES2017```中所提供的```Object.entries```方法获取对象中所有键值数组，得到的的每个键值也是元组，因为他是固定长度的。

## 18. 枚举类型

在应用开发过程中，经常会涉及到需要用某几个数值去代表某种状态，例如文章对象的状态，可以使用数字表示不同的状态，```0```代表草稿，```1```代表未发布，```2```代表已发布。

```js
const post = {
    title: 'Hello```TypeScript```',
    status: 1
}
```

也就是说状态的取值只可能是```0，1，2```这三个值，如果直接在代码中使用```0，1，2```这样的字面量表示状态的话，时间久了就可能搞不清这里的数字到底对应的是哪个状态。而且时间长了还可能混进来一些其他的值。

这种情况下使用枚举类型是最合适的。枚举类型有两个特点，第一点就是他可以给一组数值分别取上更好理解的名字，第二点是枚举中只会存在几个固定的值。并不会出现超出范围的可能性。

很多传统的编程语言中，枚举是非常常见的语言结构，不过在js中并没有这样的数据结构，很多时候都是使用对象去模拟实现枚举。

例如这里定义```posTypeScripttatus```对象，这个对象中有三个成员，分别是```draft: 0```，```unpublished: 1```，```published: 2```。可以在后面代码中使用对象中的一些属性表示文章的状态，这样的话在后面使用的过程中就不会出现刚刚所说的问题。

```js
const posTypeScripttatus = {
    draft: 0,
    unpublished: 1,
    published: 2
}
```

```TypeScript```有专门的枚举类型，可以使用```enum```关键词声明枚举。

```js
enum posTypeScripttatus = {
    draft = 0,
    unpublished = 1,
    published = 2
}
```

需要注意这里使用的是```=```而不是对象中的```:```，枚举的使用方式跟对象是一样，同样是打点调用。

枚举类型的值可以不用```=```指定，如果不指定，默认会从```0```开始累加，如果给枚举中首个成员指定了值，后面的值都会在这个基础上累加。

枚举的值除了可以是数字以外还可以是字符串，也就是字符串枚举，可以把每个值初始化为字符串，由于字符串无法像数字一样自增长，所以需要手动去给每个成员初始化明确的值。字符串枚举并不常见。

枚举类型会入侵到运行时代码，通俗一点就是会影响代码编译后的结果。```TypeScript```中使用的大多数类型经过编译转换后都会被移除掉，枚举则不会。他最终会编译为双向的键值对对象。

```js
var posTypeScripttatus;
(funcion (posTypeScripttatus) {
    posTypeScripttatus[posTypeScripttatus['draft'] = 0] = 'draft';
    posTypeScripttatus[posTypeScripttatus['unpublished'] = 1] = 'unpublished';
    posTypeScripttatus[posTypeScripttatus['published'] = 2] = 'published';
})(posTypeScripttatus || (posTypeScripttatus = {}))
```

所谓双向键值对就是可以通过键获取值，也可以通过值获取键，仔细观察这段代码会发现，无外乎就是把枚举的名称做为对象的键存储了枚举的值。然后再用值作为键存储枚举的键。这样做的目的是为了可以动态的根据枚举值获取枚举的名称。

```js
posTypeScripttatus[0]; // draft
```

如果确认代码中不会使用索引的方式访问枚举，建议使用常量枚举。常量枚举的用法就是在枚举的```enum```关键词前面加上```const```。

```ts
const enum posTypeScripttatus = {
    draft,
    unpublished,
    published
}
```

编译过后所使用的枚举会被移除掉，使用枚举值的地方会被替换为实际的值，枚举的名称会以注释的方式去标注。

## 19. 函数类型

在```JavaScript```中有两种函数定义的方式，分别是函数声明和函数表达式。

```js
function func1 (a, b) {
    return 'func1';
}
```

函数声明方式的类型约束比较简单，直接在函数每个参数后面添加对应的类型注解，返回值的类型注解添加在括号后面。如果某参数是可选的，可以使用可选参数特性，就是在参数的名称后面添加```?```。

```ts
function func1 (a: number, b: number, c?: number): string {
    return 'func1';
}

func1(100, 200);
func1(100, 200, 300);
```

也可以使用```ES6```新增的参数默认值特性，因为添加了参数默认值后，参数本身就可有可无了。

```ts
function func1 (a: number, b: number, c?: number = 10): string {
    return 'func1';
}

func1(100, 200);
```

可选参数和默认参数必须出现在参数列表的最后。因为参数会按照位置进行传递，如果可选参数出现在了必选参数的前面，这个时候必选参数是没有办法拿到正常对应的值。如果需要接收任意个数的参数，可以使用```ES6```的```reset```操作符。

```ts
function func1(...reset: number[]): string {
    return 'func1';
}
```

函数表达式所对应的类型限制也可以使用相同的方式限制函数的参数和返回值类型。

```ts
const func2 = function(a: number, b: number): string {
    return 'func2';
}
```

不过函数表达式最终是放到变量中的，接收函数的变量应该是有类型的。一般```TypeScript```能根据函数表达式推断出来变量的类型，不过如果把函数作为参数传递，也就是回调函数的方式，这种情况就必须要约束回调函数这个参数，也就是形参的类型。可以使用类似箭头函数的方式去表示。

```ts
const func2: (a: number, b: number) => string = function(a: number, b: number): string {
    return 'func2';
}
```

## 10. 任意类型

由于```JavaScript```是弱类型的，很多内置的```API```本身就支持接收任意类型的参数。```TypeScript```是基于```JavaScript```基础之上的，所以难免会在代码中需要用变量接收任意类型的数据。

定义```stringify```函数接收```value```参数，在内部使用```JSON.stringify```方法将```value```序列化成```json```字符串，这里的```value```参数就应该支持接收任意类型的参数。

```js
function stringify (value) {
    return JSON.stringify(value);
}
```

本身```JSON.stringify```就可以接收任意类型的参数，必须要有类型可以用来接收任意类型参数。

```ts
function stringify (value: any) {
    return JSON.stringify(value);
}
```

```any```就是可以用来接收任意类型数据的一种类型，需要注意的是```any```类型仍然属于动态类型。他的特点跟普通的```JavaScript```变量是一样的。也就是可以用来接收任意类型的值。而且在运行过程中还可以接收其他类型的值。

因为```any```可能存放任意类型的值，所以说```TypeScript```不会对```any```做类型检查，也就意味着可以像```JavaScript```中一样调用任意的成员。他仍然存在类型安全的问题，轻易不要使用这种类型。只有在兼容一些老的代码的时候可能会用到```any```类型。

## 21. 隐式类型推断

```TypeScript```中如果没有明确通过类型注解标记变量的类型```TypeScript```会根据这个变量的使用情况推断这个变量的类型。这种特性叫做隐式类型推断。

例如使用```let```定义```age```变量，赋值了数字，这个变量的类型就会被```TypeScript```推断为```number```类型。

```ts
let age = 18;
```

如果再去给这个变量重新赋值字符串，语法上就会出现类型错误，因为```age```已经被推断为```number```类型。

```ts
let age = 18;

age = 'string';
```

如果```TypeScript```无法去推断变量具体的类型，他会标记为```any```。

例如定义```foo```变量，声明的时候并没有为他赋值，这个时候```foo```就是```any```类型，也就是动态类型，也就是说可以在后续代码中向这个变量中放入任意类型的值。

```ts
let foo;

foo = 'string';
```

虽然```TypeScript```支持隐式类型推断，而且隐式类型推断可以简化一部分代码，但仍建议给每个变量添加明确的类型，因为这样会便于后期理解代码。

## 22. 类型断言

有些情况下```TypeScript```无法推断出变量的具体类型，作为开发者，根据代码的使用情况可以明确知道变量是什么类型。

例如下面的数组，假定这个数组是从可以明确的接口中获取到的，此时调用数组对象的```find```方法从这个数组中找出首个大于```0```的数字。很明显，这里的返回值一定是数字，因为这个数组中一定会有大于```0```的数字。

```js
const nums = [110, 120, 119, 112];

const res = nums.find(i => i > 0);
```

但是对于```TypeScript```来讲，他并不知道这些，他所推断出来这个地方的返回值是```number```或者```undefined```，因为他会认为有可能找不到。此时就不能把这个返回值当做数字使用，这种情况下就可以去断言```res```是```number```类型的。

```js
const nums = [110, 120, 119, 112];

const res = nums.find(i => i > 0);

const square = res * res;
```

断言的意思就是明确告诉```TypeScript```，你相信我，这个地方```res```一定是```number```类型。

类型断言的方式有两种，第一种是使用```as```关键词，定义```num1```变量让他等于```res as number```。此时编译器就能明确知道```num1```是数字。

```ts
const num1 = res as number;
```

另一种语法是在这个变量的前面使用```<>```，定义```num2```让他等于```<number>res```，效果仍然是一样的。

```ts
const num2 = <number>res;
```

```<>```的方式有个小问题，就是当在代码中使用了```jsx```的时候会跟```jsx```中的标签产生语法冲突。所以推荐使用```as```方式，这种用法很多时候都可以用来去辅助```TypeScript```更加明确代码中每个成员的类型。

这里还需要注意一点的是，类型断言并不是类型转换，也就是说并不是把类型转换成了另外类型。因为类型转换是代码在运行时的概念，而这个地方类型的断言只是在编译过程中的概念，当代码编译过后，断言也就不存在了，所以他跟类型转换有本质上的差异。

## 23. 接口

定义```printPost```函数接收参数```post```，打印```title```和```content```属性。

```js
function printPost (post) {
    console.log(post.title);
    console.log(post.content);
}
```

这个时候对于函数所接收的```post```对象就有一定的要求，也就是所传入的对象必须要存在```title```属性和```content```属性，只不过这种要求实际上是隐性的，没有明确的表达出来。这种情况就可以使用接口去表现这种约束。

定义接口的方式就是使用```interface```关键词，后面跟上接口的名称，可以叫做```Post```，然后是一对```{}```，里面可以添加具体的成员限制。比如```title```和```content```，类型都是```string```。

```ts
interface Post {
    title: string;
    content: string;
}
```

可以使用逗号分割成员，但是更标准的做法是使用分号分割，而且这个分号跟```js```中绝大多数的分号是一样的，可以省略。完成过后可以给```post```参数的类型设置刚刚定义的```Post```接口。

```ts
function printPost (post: Post) {
    console.log(post.title);
    console.log(post.content);
}

printPost({
    title: 'hello',
    content: 'TypeScript'
})

```

此时传入的对象必须有```title```和```content```两个成员，这就是接口的基本作用。一句话总结，接口是用来约束对象的结构的，如果对象实现接口，就必须要拥有这个接口中所约束的所有的成员。

可以编译一下这段代码，编译过后在```js```中并不会发现有任何跟接口相关的代码，也就是说```TypeScript```中的接口他只是用来为有结构的数据去做类型约束的，在实际运行阶段呢，实际这种接口他并没有意义。

## 24. 可选成员，只读成员

接口中约定的成员有一些特殊的用法，首先是可选成员，如果在对象中某成员是可有可无的，对于约束这个对象的接口来说可以使用可选成员特性。就是在成员后面添加一个```?```，比如```subTitle```成员，他的类型同样是```string```。

```ts
interface Post {
    title: string;
    content: string;
    subTitle?: string;
}
```

这种用法其实相当于给```subTitle```标记的类型是```string```或者是```undefined```。

一般来讲文章的```summary```都是从文章的内容中自动提取出来的，所以说不应该允许外界去设置他。这种情况可以使用```readonly```关键词。添加了```readonly```过后```summary```在初始化完成后不能再去修改。如果再去修改就会报错。这就是只读成员。

```ts
interface Post {
    title: string;
    content: string;
    subTitle?: string;
    readonly summary: string;
}
```

最后再来看动态成员的用法，这种用法一般适用于一些具有动态成员的对象，例如程序中的缓存对象，他在运行过程中就会出现一些动态的键值。使用```[]```，```[]```中使用```key: string```。```key```并不是固定的，可以是任意的名称，只是代表了属性的名称，他是格式，```string```就是成员名的类型，也就是键的类型。

```ts
interface Cache {
    [key: string]: string;
}
```

完成后创建```cache```对象，让他去实现这个接口，这时就可以在```cache```对象上动态的添加任意的成员了，只不过这些成员都必须是```string```类型的键值。

```ts
const cache: Cache = {};

cache.foo = 'value1';
cache.bar = 'value2'
```

## 25. 类的基本使用

类可以说是面向对象编程中最重要的概念，是用来描述一类具体事物的抽象特征。

例如手机就属于类型，这个类型的特征就是能够打电话，发信息。在这个类型下面还会有一些细分的子类，子类一定会满足父类的所有特征，然后再多出来一些额外的特征。例如智能手机，除了可以打电话发短信还能使用一些```app```。

人们是不能直接使用类的，而是去使用类的具体事物，例如手中的智能手机。

类比到程序中类也是一样的，他可以用来去描述一类具体对象的一些抽象成员，在```ES6```以前```JavaScript```是通过函数配合原型模拟实现的类。从```ES6```开始，```JavaScript```中有了专门的```class```。

在```TypeScript```中除了可以使用所有```ECMAScript```标准中类的功能，还添加了一些额外的功能和用法，例如对类成员有特殊的访问修饰符，还有一些抽象类的概念。

声明```Person```类型，在类型中声明```constructor```构造函数，构造函数中接收```name```和```age```参数，这里仍然可以使用类型注解的方式标注每个参数的类型。

构造函数的里面可以使用```this```为类型的属性赋值，不过直接去使用```this```访问当前类的属性会报错，因为当前```Person```类型上面并不存在对应的```name```和```age```。这是因为```TypeScript```中需要明确在类型中声明所拥有的一切属性。

```ts
class Person {
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
```

在类型声明属性的方式就是直接在类中定义，这个语法是```ECMAScript2016```标准定义的，可以给```name```和```gae```属性添加类型也可以通过等号直接赋初始值，不过一般情况下还是会在构造函数中动态的为属性赋值。

需要注意```TypeScript```类的属性他必须有初始值，可以在等号后面赋值也可以在构造函数中初始化，两者必须做其一，否则会报错。类的属性在使用之前必须要先在类型中声明，目的其实是为了给属性做一些类型的标注。

```ts
class Person {
    name: string = 'init name';
    age: number;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
```

除此之外仍然可以按照```ES6```标准语法为类型声明方法。方法的内部同样可以使用```this```访问当前实例对象。

```ts
class Person {
    name: string = 'init name';
    age: number;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    sayHi(msg: string): void {
        console.log(`I am ${this.name}，${msg}`);
    }
}
```

## 26. 类的访问修饰符

类中的每个成员都可以使用访问修饰符来修饰。例如给age属性添加```private```，表示```age```是私有属性，私有属性只能在类的内部访问，创建```Person```对象，可以发现```name```可以访问，```age```会报错。

```ts
class Person {
    name: string = 'init name';
    private age: number;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    sayHi(msg: string): void {
        console.log(`I am ${this.name}，${msg}`);
    }
}

const tom = new Person('tom', 18);

console.log(tom.name);
console.log(tom.age);
```

除```private```外还可以使用```public```修饰成员，意思为共有成员，在```TypeScript```中，类成员的访问修饰符默认都是```public```，加不加```public```效果都是一样的。建议手动去加上```public```的修饰符，这样代码会更加容易理解一些。

```ts
class Person {
    public name: string = 'init name';
    private age: number;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

    sayHi(msg: string): void {
        console.log(`I am ${this.name}，${msg}`);
    }
}
```

```protected```修饰符是受保护的，给```gender```属性使用```protected```，在实例对象上访问```gender```会发现是访问不到的，```protected```不能在外部直接访问。

```ts
class Person {
    public name: string = 'init name';
    private age: number;
    protected gender: boolean;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
        this.gender = true;
    }

    sayHi(msg: string): void {
        console.log(`I am ${this.name}，${msg}`);
    }
}

const tom = new Person('tom', 18);

console.log(tom.name);
// console.log(tom.age);
console.log(tom.gender);
```

定义```Student```的类型继承```Person```，在构造函数中是可以访问到父类的```gender```的。```protected```与```private```的区别是```protected```只允许在子类中访问对应的成员，```private```只能在自身类的内部访问成员。

```ts
class Student extends Person {
    constructor(name: string, age: number) {
        super(name, age); // 子类需要调用super将参数传给父类。
        console.log(this.gender);
    }
}
```

构造函数同样有访问修饰符默认也是```public```，如果说设置为```private```这个类型就不能在外部被实例化也不能被继承，在这样一种情况下，就只能够在这个类的内部添加静态方法，然后在静态方法中创建类型的实例。

```static```是```ES6```标准定义的，可以在```create```方法中使用```new```创建类型实例，在外部使用```create```静态方法获取```Student```类型的对象。

```ts
class Student extends Person {
    private constructor(name: string, age: number) {
        super(name, age); // 子类需要调用super将参数传给父类。
        console.log(this.gender);
    }

    static create(name: string, age: number) {
        return new Student(name, age);
    }
}

const jack = Student.create('jack', 18);
```

如果把构造函数标记为```protected```类型是不能在外面被实例化的，相比于```private```他是允许继承的。

## 27. 类的只读属性

除```private```和```protected```还可以使用```readonly```关键词将成员设置为只读。需要注意如果属性已经有了访问修饰符，```readonly```应该跟在后面，只读属性可以在类型声明的时候通过等号的方式初始化，也可以在构造函数中初始化，二者只能选其一。也就是说不能在声明的时候初始化，然后在构造函数中再去修改它，因为这样破坏了```readonly```的规则。

```ts
class Person {
    public name: string = 'init name';
    private age: number;
    protected readonly gender: boolean;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
        this.gender = true;
    }

    sayHi(msg: string): void {
        console.log(`I am ${this.name}，${msg}`);
        console.log(this.age);
    }
}

const tom = new Person('tom', 18);

console.log(tom.name);
```

初始化过后```gender```属性就不允许再被修改了，无论是在内部还是外部，都是不允许修改的。

## 28. 类与接口

相比于类来说接口的概念更为抽象一点。比如说手机是类型，实例后的类型都是能够打电话，发短信的，因为手机类的特征就是打电话和发短信。但是能够打电话的不仅仅只有手机，还有座机，只是座机并不属于手机类目，而是单独的类目，因为他不能发短信，也不能拿着到处跑。

在这种情况下就会出现，不同的类之间会有一些共同的特征，对于这些公共的特征一般会使用接口来抽象。可以理解为手机之所以可以打电话是因为他实现了能够打电话的协议，座机也能够打电话是因为他也实现了这个协议。

这里所说的协议在程序中就叫做接口，当然如果是第一次接受这种概念的话，可能理解起来有些吃力，个人的经验就是多思考，多从生活的角度去想，如果实在想不通，更粗暴的办法就是不断的去用，用的过程中慢慢的去总结规律，时间长了自然也就理解了。

这里定义两个类型，分别是Person和Animal，也就是人类和动物类，他们实际上是两个完全不同的类型，但是他们之间也会有一些相同的特性，例如他们都会吃东西都会跑。

```ts
class Person {
    eat (food: string): void {}
    run (distance: number) {}
}

class Animal {
    eat (food: string): void {}
    run (distance: number) {}
}
```

这种情况就属于不同的类型实现了相同的接口，可能有人会问，为什么不给他们之间抽象公共的父类，然后把公共的方法定义到父类中。这个原因很简单，虽然人和动物都会吃，都会跑，但是说人吃东西和狗吃东西能是一样的么，他们只是都有这样的能力，而这个能力的实现肯定是不一样的。

在这种情况下就可以使用接口去约束这两个类型之间公共的能力，定义接口```EatAndRun```，然后在接口中分别添加```eat```和```run```两个方法的约束。这里需要使用函数签名的方式去约束这两个方法的类型，而这不做具体的方法实现。接口就是只约定类型不做实现。

```ts
interface EatAndRun {
    eat (food: string): void;
    run (distance: number): void;
}
```

有了接口使用```implements```来实现```EatAndRun```接口，此时在类型中就必须要有对应的成员，如果没有就会报错，因为实现接口就必须实现接口的成员。

```ts
class Person implements EatAndRun {
    eat (food: string): void {}
    run (distance: number) {}
}
```

需要注意在```C#```和```Java```等语言中建议尽可能让每个接口的定义更加简单，更加细化。```EatAndRun```接口中抽象了两个方法，就相当于抽象了两个能力，正确的做法应该是把这个接口拆成```Eat```接口和```Run```接口，每个接口只有单个成员。然后就可以在类型的后面使用逗号同时实现```Eat```和```Run```两个接口。

```ts
interface Eat {
    eat (food: string): void;
}

interface Run {
    run (distance: number): void;
}

class Person implements Eat, Run {
    eat (food: string): void {}
    run (distance: number) {}
}
```

这里多说一句题外话，就是大家千万不要把自己框死在某一门语言或者是技术上面，最好可以多接触，多学习一些周边的语言或者技术，因为这样的话可以补充你的知识体系。最简单来说，只了解```JavaScript```的开发人员，即便说他对```JavaScript```再怎么精通，也不可能设计出一些比较高级的产品。例如现在比较主流的一些框架，他们大都采用一些MVVM这样的思想，这些思想实际上最早是出现在微软的WPS技术中的，如果你有更宽的知识面，你可以更好的把多家的思想融合到一起，所以说视野应该放宽一些。

## 29. 抽象类

抽象类在某种程度上来说跟接口有点类似，也是用来约束子类中必须要有某些成员。不同于接口的是，抽象类可以包含一些具体的实现，而接口只能是成员的抽象，并不包含具体实现。

一般比较大的类目都建议大家使用抽象类，例如刚刚所说的动物类，其实就应该是抽象的，因为所说的动物只是泛指，并不够具体，在他的下面一定会有一些更细化的划分，比如说小狗，小猫之类的。而且在生活中一般都会说买了一条狗，或者说买了一只猫，从来没有人说买了动物。

```ts
abstract class Animal {
    eat (food: string): void {}
}
```

定义抽象类的方式就是在```class```前面添加```abstract```，这样类型就被定义成了抽象类，他只能被继承，不能实例化。必须使用子类继承抽象类，比如定义Dog类型，让他继承Animal，抽象类中还可以定义一些抽象方法，抽象方法可以使用```abstract```关键词修饰，定义run的抽象方法，需要注意的是抽象方法不需要方法体。父类中有抽象方法时，子类必须要实现这个方法。

```ts
abstract class Animal {
    eat (food: string): void {}
    abstract run (distance: number): void;
}

class Dog extends Animal {
    run (distance: number): void {}
}
```

此时使用这个子类创建对象时，会同时拥有父类中的一些实例方法以及自身所实现的方法。这就是抽象类的基本使用。关于抽象类更多的还是去理解他的概念，在使用上并没有什么复杂的地方。

## 30. 泛型

泛型（```Generics```）是指在定义函数、接口或类的时候，不预先指定其类型，而是在使用时手动指定其类型的一种特性。比如创建函数，这个函数会返回任何它传入的值。这段代码编译不会出错，但是存在显而易见的缺陷，就是没有办法约束输出的类型与输入的类型保持一致。

```ts
function identity(arg: any): any {
  return arg
}

identity(3) // 3
```

可以使用泛型来解决这个问题。

```ts
function identity<T>(arg: T): T {
  return arg
}

identity(3) // 3
```

在函数名后面加了```<T>```，其中```T```表示任意输入的类型，后面的```T```表示输出的类型，且与输入保持一致。当然也可以在调用时手动指定输入与输出的类型。

```ts
identity<number>(3) // 3
```

在泛型函数内部使用类型变量时，由于事先并不知道它的类型，所以不能随意操作它的属性和方法。比如类型```T```上不一定存在```length```属性，所以编译的时候就报错了。

```ts
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length)   // err 
  return arg
}
```

可以对泛型进行约束，传入的值约束必须包含```length```的属性，也就是泛型约束。

```ts
interface lengthwise {
  length: number
}

function loggingIdentity<T extends lengthwise>(arg: T): T {
  console.log(arg.length)   // err 
  return arg
}

loggingIdentity({a: 1, length: 1})  // 1
loggingIdentity('str') // 3
loggingIdentity(6) // err  传入是参数中未能包含 length 属性
```

多个参数时也可以在泛型约束中使用类型参数，比如声明了类型参数，它被另一类型参数所约束。如果想用属性名从对象中获取属性，并且还需确保属性存在于对象上，这就需要在两个类型之间使用约束。

简单举例来说就是定义函数，接收两个参数，首个是个对象```obj```，第二个参数是第一参数里面的键名```key```， 需要输入```obj[key]```。

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key]
}

let obj = { a: 1, b: 2, c: 3 }

getProperty(obj, 'a') // success
getProperty(obj, 'm') // err obj 中不存在 m 这个参数
```

还可以为泛型中的参数指定默认类型，当使用泛型时如果没有在代码中直接指定参数类型，实际值参数中也无法推测出类型时，默认类型就会起作用。

```ts
function createArr<T = string>(length: number, value: T): Array<T> {
  let result: T[] = []
  for( let i = 0; i < lenght; i++ ) {
    result[i] = value
  }
  return result
}
```

简单来说，泛型就是把定义时不能明确的类型变成参数，在使用的时候再去传递这个类型。

## 31. 类型声明

项目开发过程中经常会用到一些第三方模块，这些模块并不都是通过```TypeScript```编写的，所提供的成员也不会有强类型的体验。比如```lodash```模块，模块中就提供了很多工具函数，导入的时候```TypeScript```就已经报出了错误，找不到类型声明的文件。

提取一下```camelCase```函数，这个函数的作用是把字符串转换成驼峰格式，他的参数应该是```string```，返回值也应该是```string```，但是在调用的时候并没有任何的类型提示。

```ts
import { camelCase } from 'lodash';

const res = camelCase('hello typed');
```

在这种情况下就需要单独的类型声明了，可以使用```declare```语句声明这里的函数类型，具体的语法就是```declare function```后面跟上函数名称，参数是```input```类型是```string```，返回值也是```string```。

```ts
declare function camelCase (input: string ): string;
```

有了声明过后，再去使用```camelCase```函数就会有对应的类型限制了。这里所谓的类型声明，说白了就是成员他在定义的时候因为种种原因没有声明明确的类型，在使用的时候可以单独为他再做一次声明。

这种用法就是为了考虑兼容一些普通的```js```模块，由于```TypeScript```的社区非常强大，目前比较常用的```npm```模块都已经提供了对应的声明，只需要安装一下类型声明模块就可以了。比如```lodash```的报错模块提示需要安装```@types/lodash```的模块，这个模块就是```lodash```对应的类型声明模块。```ts```中```.d.ts```文件就是做类型声明的文件。

除了类型声明模块，现在越来越多的模块已经在内部继承了声明文件，很多时候已经不需要单独安装声明模块了。例如```query-string```模块，作用就是解析```url```中的```query-string```字符串，这个模块就已经包含了类型声明文件，直接导入的对象就直接会有类型约束。

在```TypeScript```中引用第三方模块，如果这个模块中不包含对应的类型声明文件，可以尝试安装所对应的类型声明模块，类型声明模块一般就是```@types/模块名```格式的。如果没有对应的类型声明模块，就只能自己使用```declare```语句声明对应的模块类型了。对于```declare```详细的语法这里不再单独介绍，有需要的可以单独查询官方文档。
