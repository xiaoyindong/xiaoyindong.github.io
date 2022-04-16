## 1. 概述

首先要考虑一下该如何去精准测试```JavaScript```代码的性能，精准测试本质就是经过大量的数据采集，然后执行样本的数学统计分析，从而得出比对的结果来，证明什么样的脚本执行效率更高。

对编码者来说可能更多的只是关注该如何使用脚本去实现某一个功能，而不是去做大量的数学统计。这里采用基于```Benchmark.JavaScript```的一个```perf```网站，进行在线的```JavaScript```脚本的性能测试。

## 2. perf使用流程

1. 使用GitHub账号登录

2. 填写个人信息(非必须)

3. 填写详细的测试用例信息(title, slug)

这里比较关注的是测试用例的```title```和```slug```，会用它去生成一个短连接，用于去其他的地方访问测试用例。

4. 填写准备代码(DOM操作时经常使用)

5. 填写必要的setup和teardown代码

```setup```可以理解为是当前要做的前置准备，比如说要使用手机就要先打开手机，```teardown```就是所有代码执行完之后要做的销毁操作，比如使用数据库的时候，用完了应该把当前的链接资源释放掉让内存得到释放。

6. 填写测试代码片段

这里可以填一个片段也可以填多个片段，取决于想要测试几个片段，有了这些操作以后就可以直接在浏览器中运行脚本了。

最终当脚本在网站上执行完成之后就会给出数据上的体现，通过数据就可以得出，什么样类型的```JavaScript```脚本会具有更高的执行效率。可以在浏览器当中测试使用一下。

首先打开```JavaScriptperf```(```www.JavaScriptperf.com```)网站，登录完成之后，进入到填写个人信息界面，这里是非必填的，向下可以发现```Test case details```栏目。

这里有两个是必填的，一个是```title```，一个是```slug```，需要注意的是```slug```必须是唯一的，因为他会去生成一个空间，利于访问自己的测试用例。

往下看有个叫做```Preparation code html```栏目，这里就是准备代码，也就是需要用到一些```DOM```操作或者说需要引入一些第三方的资源库的时候，可以在这个里边贴那些代码。

在下边是```setup```和```teardown```的填写区域，可以在后续的测试中有所应用，现在不需要关心。

再往下就是当前代码片段的填写，这里有几个是必填的，第一个是测试标题，接着是要测试的代码片段，可以直接把代码贴在这里面，然后这里有多个片段，还可以自己添加。

准备好这些内容之后，可以直接保存，保存之后会跳转到新的界面。

## 3. 慎用全局变量

程序执行过程中如果针对某些数据需要进行存储，尽可能放置在局部作用域变成局部变量。在全局的范围内定义变量是存在全局的执行上下文中的，这个上下文是后续程序查找数据过程中所有作用域链的最顶端。按查找的过程来说，某些局部作用域没有找到的变量最终都会查找到顶端的全局作用域。这种情况下查找的时间消耗是非常大的，也降低了代码的执行效率。

除此以外全局上下文中定义的变量是存活于上下文执行栈的。上下文执行栈直到程序退出才会消失，这对GC工作是非常不利的，因为只要```GC```发现变量处于存活状态就不会把他当做垃圾对象进行回收。这样的做法会降低程序运行过程中内存的使用。

除此以外，如果某个局部作用域中定义了同名的变量，有可能造成变量的命名污染，将全局的数据进行了遮蔽。总归来说使用全局变量的时候需要考虑更多，否则就会带来意想不到的情况。

这里针对具体的场景来判断使用全局变量和使用局部变量的执行效率，二者进行对比。首先定义两个全局变量，```i```和```str```，```str```赋值个空的字符串。接下来通过```for```循环生成很长的字符串，这个字符串建议小一些，不然机器可能会出现问题。在循环内部拼接字符串的长度。这是采用全局作用域的方式实现。

```js
var i, str = '';
(function() {
    for (i = 0; i < 1000; i++) {
        str += i;
    }
})()
```

另外一个和第一个类似，不过使用的是局部变量```i```和```str```。

```js
(function() {
    let str = '';
    for (let i = 0; i < 1000; i++) {
        str += i;
    }
})()
```

这个时候就相当是用两份不同的代码去完成了一个相同的效果。接下来把他们放在```JavaScriptperf```当中大量执行，从而得出谁的执行效率更高。

在```JavaScriptperf```中分别贴入这两段代码，运行脚本得出最终的性能结果。执行完成以后可以发现，采用全局变量和采用局部变量之间的差距是非常大的。

## 4. 缓存全局变量

缓存全局变量指的是程序执行过程中，全局变量的使用如果是无法避免的，比如```document```，可以选择将需要大量使用的全局变量放置在某一个局部作用域中，从而达到一种缓存效果。这里比较一下使用缓存和不使用缓存的性能差异。

```html
<body>
    <input value="btn" id="btn1" />
    <input value="btn" id="btn2" />
    <input value="btn" id="btn3" />
    <input value="btn" id="btn4" />
    <p>111</p>
    <input value="btn" id="btn5" />
    <input value="btn" id="btn6" />
    <p>222</p>
    <input value="btn" id="btn7" />
    <input value="btn" id="btn8" />
    <p>333</p>
    <input value="btn" id="btn9" />
    <input value="btn" id="btn10" />
    <script>
        function getBtn() {
            let oBtn1 = document.getElementById('btn1');
            let oBtn3 = document.getElementById('btn3');
            let oBtn5 = document.getElementById('btn5');
            let oBtn7 = document.getElementById('btn7');
            let oBtn9 = document.getElementById('btn9');
        }

        function getBtn2() {
            let obj = document;
            let oBtn1 = obj.getElementById('btn1');
            let oBtn3 = obj.getElementById('btn3');
            let oBtn5 = obj.getElementById('btn5');
            let oBtn7 = obj.getElementById('btn7');
            let oBtn9 = obj.getElementById('btn9');
        }
    </script>
</body>
```

```JavaScriptperf```结果可以发现，使用缓存的效果比没使用缓存会有一些优势。

## 5. 通过原型对象添加附加方法

```JavaScript```中存在三种概念，构造函数，第原型对象，实例对象。实例对象和构造函数都是可以指向原型对象的，可以把某些调用频繁的方法添加在原型对象上，而不需要放在构造函数内部。比较一下两种不同的实现方式的性能。

```js
var fn1 = function() {
    this.foo = function() {
        console.log(11111);
    }
}

let f1 = new fn1();

var fn2 = function() {}
fn2.prototype.foo = function() {
    console.log(11111);
}

let f2 = new fn2();
```

```JavaScriptperf```对比发现构造函数添加的方法相比通过原型添加的方法效率要低很多。

## 6. 避开闭包陷阱

闭包在```JavaScript```中是非常强大的语法，可以带来非常大的便捷，不过闭包如果使用不当很容易造成内存泄漏，不要为了闭包而闭包。

这里对比一下使用闭包和使用函数重用哪种方式执行效率更高。首先定义函数，接收另外一个函数作为参数。

```js
function test(func) {
    console.log(func());
}

function test2() {
    var name = 'yd';
    return name;
}

test(function() { // 使用闭包
    var name = 'yd';
    return name;
});

test(test2) // 不使用闭包
```

## 7. 避免属性访问方法使用

属性访问方法是跟面向对象语法相关的，为了更好的实现封装性，更多的时候可能将对象的成员属性和方法放在函数内部，然后对外部暴露方法对这些属性进行增删改查。但是这个特性在```JavaScript```中并不是那么的适用，在```JavaScript```里是不需要属性的访问方法的，所有的属性在外部都是可见的，使用属性访问方法的时候他相当是增加了一层重定义，对当前的访问控制来说没有太多的意义。所以不推荐属性访问方法的使用。

```js
function Person() {
    this.name = 'icoder';
    this.age = 18;
    this.getAge = function() {
        return this.age;
    }
}

const p = new Person();
// 使用访问方法
const a = p.getAge();

function Person() {
    this.name = 'icoder';
    this.age = 18;
    this.getAge = function() {
        return this.age;
    }
}
const p = new Person();
// 不使用访问方法
const b = p.age;
```

对比发现通过属性访问方法比直接访问效率要低很多。

## 8. For循环优化

使用```for```循环遍历数组的时候，不要直接使用数组的属性，最好缓存下来。

```js
var arrList = [];
arrList[10000] = 'icoder';
for (var i = 0; i < arrList.length; i++) {
    console.log(arrList[i])
}

// 缓存数组长度
for (var i = arrList.length; i; i--) { 
    console.log(arrList[i])
}
```

## 9. 选择最优的循环方法

对比```forEach```，```for```，```for ... in```三种方式谁的效率更高一些。

```js
var arrList = new Array(1, 2, 3, 4, 5)

arrList.forEach(function(item) {
    console.log(item);
})

for (var i = arrList.length; i; i--) { // 缓存数组长度
    console.log(arrList[i])
}

for (var i in arrList) { // 缓存数组长度
    console.log(arrList[i])
}
```

```foreach```效率最高，```for...in...```效率最低。

## 10. 文档碎片优化节点添加

```DOM```操作是非常消耗性能的，特别是创建新的节点，将它添加至界面中时，这个过程一般都会伴随着回流和重绘。这两个操作对性能的消耗又是比较大的。

```js
// 不使用优化
for (var i = 0; i < 10; i++) {
    var oP = document.createElement('p');
    oP.innerHTML = i;
    document.body.appendChild(oP);
}

// 使用优化
const fragEle = document.createDocumentFragment();
for (var i = 0; i < 10; i++) {
    var oP = document.createElement('p');
    oP.innerHTML = i;
    fragEle.appendChild(oP);
}
document.body.appendChild(fragEle);
```

## 11. 克隆优化节点操作

页面渲染完成之后页面中存在很多```p```标签，当要新增节点的时候，可以找到一个与新增节点类似的已经存在于页面中的节点，通过克隆的方式添加到界面当中。这样优化的内容是本身已经具有的样式和属性就不需要后续执行添加了，这就是所谓的优化。

```html
<body>
    <p id="box1">old</p>
    <script>
        // 创建方式
        for (var i = 0; i < 10; i++) {
            var oP = document.createElement('p');
            oP.innerHTML = i;
            document.body.appendChild(oP);
        }
        // 克隆方式
        var oldP = document.getElementById('box1');
        for (var i = 0; i < 10; i++) {
            var newP = oldp.cloneNode(false);
            newP.innerHTML = i;
            document.body.appendChild(newP);
        }
    </script>
</body>
```

## 12. 直接量替换newObject

定义对象和数组的时候，有两种不同的形式，可以使用```new```的方式获取相应的数据，也可以直接采用字面量。

```js
// 字面量方式
var a = [1, 2, 3];

// 实例化方式
var a1 = new Array(3)
a1[0] = 1;
a1[1] = 2;
a1[2] = 3;
```
