## let 与块级作用域

在```ES2015```之前，```ECMAScript```当中只有全局作用域和函数作用域两种类型的作用域。```ES2015```中新增了块级作用域。

块指的就是代码中用一对```{}```所包裹起来的范围，例如```if```语句和```for```语句中的```{}```都会产生这里所说的块。

```js
if (true) {
    consoel.log('yd');
}

for (var i = 0; i < 10; i++) {
    console.log('yd');
}
```

在快级内定义的成员，外部是无法访问的。

这样一个特性非常适合声明```for```循环当中的计数器。

```js
for (let i = 0; i < 3; i++) {
    for (let i = 0; i < 3; i++) {
        console.log(i); 
    }
}
```

## const

特点就是在```let```的基础上多了一个只读特性，一旦声明过后就不能够再被修改，如果在声明过后再去修改这个成员就会出现错误。

```js
const name = 'yd';

name = 'zd'; // 错误
```

这里要注意的问题```const```所声明的成员不能被修改，并不是说不允许修改恒量中的属性成员。下面这种情况是允许的。

```js
const obj = {}
obj.name = 'yd';
```

## 数组的解构

```js
const arr = [100, 200, 300];
const [foo, bar, baz] = arr;
console.log(foo, bar, baz);
```

如果只是想获取其中某个位置所对应的成员，例如只获取第三个成员, 这里可以把前两个成员都删掉。但需要保留对应的逗号。

```js
const [, , baz] = arr;
console.log(baz);
```

可以在解构位置的变量名之前添加```...```表示提取从当前位置开始往后的所有成员，最终所有的结果会放在一个数组当中。

```js
const [foo, ...rest] = arr;
console.log(rest);
```

三个点的用法只能在解构位置的最后一个成员上使用，另外如果解构位置的成员个数小于被解构的数组长度，就会按照从前到后的顺序去提取，多出来的成员就不会被提取。反之如果解构位置的成员大于数组长度，那么提取到的就是```undefined```。这和访问数组当中一个不存在的下标是一样的。

```js
const [foo, bar, baz, more] = arr;
console.log(more); // undefined
```

如果需要给提取到的成员设置默认值，这种语法也是支持的，只需要在解构变量的后面跟上一个等号，然后后面写上一个默认值。

```js
const [foo, bar, baz, more = 'default value'] = arr;
console.log(more);
```

## 对象的解构

对象的结构需要去根据属性名去匹配提取。

```js
const obj = { name: 'yd', age: 18 };
```

如果有这个名称就会产生冲突。这个时候可以使用重命名的方式去解决这个问题。

```js
const obj = { name: 'yd', age: 18 };

const { name: name1 } = obj;

console.log(name1);
```

## 模板字符串

```ES2015```新增了模板字符串，使用反引号\`声明，。如果在字符串中需要使用\`，可以使用斜线去转译。

在模板字符串当中可以支持多行字符串。

```js
const str = `
123
456
`
```

模板字符串当中还支持通过插值表达式的方式在字符串中去嵌入所对应的数值，在字符串中可以使用```${name}```就可以在字符串当中嵌入```name变```量中的值。

```js
const name = 'yd';
const age = 18;

const str = `my name is ${name}, I am ${age} years old`;
```

在定义模板字符串的时候可以在前面添加一个标签，标签实际上是一个特殊的函数，会调用这个函数。 

```js
const name = 'yd';
const age = 18;

const result = tag`My name is ${name}, I am ${age} years old`;
```

函数可以接收到一个数组参数，是模板字符串内容分割过后的结果。

```js
const tag = (params) => {
    consoel.log(params); // ['My name is ', ' I am ', ' years old'];
}
```

除了这个数组以外，还可以接收到所有在这个模板字符串中出现的表达式的返回值。

```js
const tag = (params, name, age) => {
    consoel.log(params, name, age); // ['My name is ', ' I am ', ' years old']; 'yd' 18
}

const str = tag`hello ${'world'}`;

```

## 字符串扩展方法

如果想要知道这个字符串是否以```Error```开头。

```js
console.log(message.startsWith('Error')); // true
```

同理如果想要知道这个字符串是否以```.```结尾。

```js
console.log(message.endsWith('.')); // true
```

如果需要明确的是字符串中间是否包含某个内容。

```js
console.log(message.includes('foo')); // true
```

## 函数参数默认值

可以直接在形参的后面直接通过等号去设置一个默认值。

```js
function foo (enable = true) {
    console.log(enable); // false
}

foo(false);
```

如果有多个参数的话，带有默认值的这种形参一定要出现在参数列表的最后。

```js
function foo (bar, enable = true) {
    console.log(enable); // false
}

foo(false);
```

## 剩余参数

接收从当前这个参数的位置开始往后所有的实参。

```js

// function foo() {
//     console.log(arguments); // 参数集合
// }

function foo (...args) => {
    console.log(args); // 参数集合
}

foo(1, 2, 3, 4);
```

这种操作符只能出现在形参列表的最后一位，并且只可以使用一次。

## 展开数组

```js
console.log( ...arr );
```

## 箭头函数

```js
function inc (number) {
    return number + 1;
}

const inc = n => n + 1;
```

此时你会发现，相比于普通的函数，剪头函数确实大大简化了所定义函数这样一些相关的代码。

剪头函数的左边是参数列表，如果有多个参数的话可以使用```()```包裹起来，剪头的右边是函数体。如果在这个函数的函数体内需要执行多条语句，同样可以使用```{}```去包裹。如果只有一句代码可以省略```{}```。

```js
const inc = (n , m) => {
    return  n + 1;
};
```

箭头函数不会改变```this```指向，剪头函数当中没有```this```的机制，外面```this```是什么，在里面拿到的就是什么，任何情况下都不会发生改变。

```js
const person = {
    name: 'yd',
    sayHi: () => {
        console.log(this.name);
    }
}

person.sayHi(); // undefined
```

## 对象字面量的增强

如果变量名与添加到对象中的属性名是一样的，可以省略掉```:```变量名。

```js
const bar = '123';
const obj = {
    bar
}
```

除此之外如果需要为对象添加一个普通的方法，现在可以省略里面的```:function```。

```js
const obj = {
    method1 () {
        console.log('method1');
    }
}

console.log(obj)
```

## 动态属性名

```js
const obj = {
    [Math.random()]: 123,
}
```

## Object.assign

将多个源对象当中的属性复制到一个目标对象当中，如果对象当中有相同的属性，那么源对象当中的属性就会覆盖掉目标对象的属性。


第一个参数就是目标对象，所有源对象当中的属性都会复制到目标对象当中，返回值是目标对象。

```js

const result = Object.assign(target, source1);

console.log(target, result === target); {a: 123, c: 456, b: 123 }// true
```

```Object.assign```用来为```options```对象参数设置默认值也是一个非常常见的应用场景。

## Object.is

判断两个值是否相等，正负零就可以被区分开，而且```NaN```也是等于```NaN```的。

```js
Object.is(+0, -0); // false
Object.is(NaN, NaN); // true
```

## Proxy

为对象设置访问代理器，可以轻松监视到对象的读写过程，相比于````defineProperty````，```Proxy```的功能要更为强大甚至使用起来也更为方便。

```js
const person = {
    name: 'yd',
    age: 18
}

const personProxy = new Proxy(person, {
    get() {},
    set() {}
})
```


```Object.defineProperty```只能监听到对象属性的```读取```或```写入```，```Proxy```除读写外还可以监听对象中属性的```删除```，对对象当中```方法调用```等。

```js
const person = {
    name: 'yd',
    age: 18
}
const personProxy = new Proxy(person, {
    deleteProperty(target, property) {
        console.log(target, property);
        delete target[property];
    },
})
```

get: 读取某个属性

set: 写入某个属性

has:```in```操作符调用

defineProperty:```Object.defineProperty()```

getOwnPropertyDescriptor:```Object.getOwnPropertyDescriptor()```

deleteProperty:```delete```操作符调用

ownKeys:```Object.keys()```,```Object.getOwnPropertyNames()```,```Object.getOwnPropertSymbols()```

getPropertyOf:```Object.getPropertypeOf()```

setPropertyOf:```Object.setProtoTypeOf()```

isExtensible:```Object.isExtensible()```

preventExtensions:```Object.preventExtensions()```

apply: 调用一个函数

construct: 用```new```调用一个函数。

```Proxy```可以直接监视数组的变化。

```js
const list = [];
const listProxy = new Proxy(list, {
    set(target, property, value) {
        console.log(target, property, value);
        target[property] = value;
        return true; // 写入成功
    }
});

listProxy.push(100);
```

```Proxy```内部会自动根据```push```操作推断出来他所处的下标，每次添加或者设置都会定位到对应的下标```property```。

```Proxy```是以非入侵的方式监管了对象的读写，那也就是说一个已经定义好的对象不需要对对象本身去做任何的操作，就可以监视到他内部成员的读写，而```defineProperty```的方式就要求必须按特定的方式单独去定义对象当中那些被监视的属性。

## Reflect

```Reflect```属于一个静态类，不能通过```new```构建实例对象。只能够去调用静态类中的静态方法,类似```Math```对象。

```Reflect```封装了一系列对对象的底层操作，方法的方法名与```Proxy```的处理对象里面的方法成员是完全一致的。其实这也是````Proxy````处理对象那些方法内部的默认实现.

```js
const obj = {
    foo: '123',
    bar: '456',
}

const Proxy = new Proxy(obj, {
    get(target, property) {
        console.log('实现监视逻辑');
        return Reflect.get(target, property);
    }
})
```

```Reflect```最大的意义是提供了一套统一操作```Object```的```API```，因为之前操作对象的方法太乱了，```Reflect```统一了对象的操作方式。

## Promise

```Promise```提供了一种全新的异步编程解决方案，通过链式调用的方式解决了在传统异步编程过程中回调函数嵌套过深的问题。

## class类

与一些老牌面向对象语言中```class```非常相似，如果需要在构造函数当中做一些额外的逻辑，可以添加一个```constructor```方法，这个方法就是构造函数。同样可以在这个函数中使用```this```去访问当前类型的实例对象。

```js
class Person {
    constructor (name) {
        this.name = name;
    }
    say() {
        console.log(this.name);
    }
}
```

类型中的方法分为```实例方法```和```静态方法```，实例方法就是需要通过这个类型构造的实例对象去调用，静态方法是直接通过类型本身去调用。可以通过static关键字定义。

```js
class Person {
    constructor (name) {
        this.name = name;
    }
    say() {
        console.log(this.name);
    }
    static create (name) {
        return new Person(name);
    }
}
```

调用静态方法是直接通过类型然后通过成员操作符调用方法名字，因为静态方法是挂载到类型上面的，所以说在静态方法内部他不会指向某一个实例对象，而是当前的类型。


```js
const yd = Person.create('yd');
```

可以通过关键词```extends```实现继承，super对象指向父类, 调用它就是调用了父类的构造函数。

```js
class Student extends Person {
    constructor(name, number) {
        super(name);
        this.number = number;
    }
    hello () {
        super.say();
        console.log(this.number);
    }
}

const s = new Student('yd', '100');

s.hello();
```

## Set

与传统的数组非常类似，不过```Set```内部的成员是不允许重复的。也就是说每一个值在同一个```Set```中都是唯一的。可以通过这个实例的```add```方法向集合当中去添加数据，由于```add```方法他会返回集合对象本身，所以可以链式调用。如果在这个过程中添加了之前已经存在的值那所添加的这个值就会被忽略掉。

```js
const s = new Set();

s.add(1).add(2).add(3).add(2);
```

可以使用集合对象的```forEach```方法遍历。

```js
s.forEach(i => console.log(i));
```

也可以使用```for...of```循环。

```js
for (let i of s) {
    console.log(i);
}
```

通过```size```属性来去获取整个集合的长度。

```js
console.log(s.size);
```

```has```方法就用来判断集合当中是否存在某一个特定的值。

```js
console.log(s.has(100)); // false
```

```delete```方法用来删除集合当中指定的值，删除成功将会返回一个```true```。

```js
console.log(s.delete(3)); // true
```

```clear```方法用于清除当前集合当中的全部内容。

```js
s.clear()
```

## Map

```Map```类型算是严格上的键值对类型，用来映射两个任意类型之间键值对的关系。可以使用```set```方法存数据。键可以是任意类型的数据。不需要担心他会被转换为字符串。

```js
const m = new Map();

const key = {};

m.set(key, 18);

console.log(m);
```

```get```方法获取数据，```has```方法判断他里面是否存在某个键。```delete```方法删除某个键。```clear```方法清空所有的键值。

```js
console.log(m.get(key));

console.log(m.has(key));

m.delete(key);

m.clear();
```

可以使用forEach方法遍历，回调函数当中第一个参数就是被遍历的值，第二个参数是被遍历的键。

```js
m.forEach((value, key) => {
    console.log(value, key);
})
```

## Symbol

```ES2015```提供了一种全新的原始数据类型```Symbol```，翻译过来的意思叫做符号，翻译过来就是表示一个独一无二的值。

通过```Symbol```函数可以创建```Symbol```类型的数据，```typeof```的结果也是```symbol```。

```js
const s = Symbol();
typeof s; // symbol类型
```

```Symbol```创建时允许接收一个字符串，作为这个值的描述文本, 对于多次使用```Symbol```时就可以区分出是哪一个```Symbol```，这个参数仅是描述作用，相同的描述字段生成的值仍是不同的。

```js
const s1 = Symbol('foo');
const s2 = Symbol('foo');

s1 === s2; // false
```

对象允许使用```Symbol```作为属性名。那也就是说现在对象的属性名可以是两种类型```string```和```Symbol```。

```js
const person = {
    [Symbol()]: 123,
    [Symbol()]: 456
}
```

```Symbol```除了用在对象中避免重复以外，还可以借助这种类型的特点来模拟实现对象的私有成员。以前私有成员都是通过约定，例如约定使用下划线开头就表示是私有成员。约定外界不允许访问下划线开头的成员。

现在有了```Symbol```就可以使用```Symbol```去作为私有成员的属性名了。在这个对象的内部可以使用创建属性时的```Symbol```。去拿到对应的属性成员。

```js
const name = Symbol();
const person = {
    [name]: 'yd',
    say() {
        return this[name];
    }
}
```

```Symbol```类型提供的一个静态方法```for```，这个方法接收一个字符串作为参数，相同的参数一定对应相同的值。

```js
const s1 = Symbol.for('foo');
const s2 = Symbol.for('foo');

s1 === s2; // true
```

这个方法维护了一个全局的注册表，为字符串和```Symbol```提供了一个对应关系。需要注意的是，在内部维护的是字符串和```Symbol```的关系，那也就是说如参数不是字符串，会转换为字符串。

```js
const s1 = Symbol.for('true');
const s2 = Symbol.for(true);

s1 === s2; // true
```

```Symbol```内部提供了很多内置的```Symbol```常量，用来去作为内部方法的标识，可以让自定义对象去实现一些```js```内置的接口。

如果想要自定义对象的toString标签，ECMAScript要求使用Symbol的值来去实现这样一个接口。

```js
obj[Symbol.toStringTag] = 'test'
obj.toString(); // [object test];
```

使用```Symbol```的值作为对象的属性名那这个属性通过传统的```for in```循环是无法拿到的。而且通过```Object.keys```方法也是获取不到这样```Symbol```类型的属性名。```JSON.stringify```去序列化，```Symbol```属性也会被隐藏掉。
```js
const obj = {
    [Symbol()]: 'symbol value',
    foo: 'normal value'
}

for (var key in obj) {
    console.log(key);
}

Object.keys(obj);
```

总之这些特性都使得```Symbol```属性，特别适合作为对象的私有属性，当然想要获取这种类型的属性名可以使用```Object.getOwnPropertySymbols(obj)```方法。

```js
Object.getOwnPropertySymbols(obj)
```

## for...of

新增的遍历方式，未来会作为遍历所有数据结构的统一方式。
```js
const arr = [1, 2, 3, 4];
for (const item of arr) {
    console.log(item); // 1， 2，3，4
    // break; // 终止循环
}
```

```for...of```循环可以使用```break```关键词随时去终止循环。除了数组可以直接被```for...of```循环去遍历，一些伪数组对象也是可以直接被```for...of```去遍历的，例如```arguments```，```set```，```map```。

```for...of```在遍历```Map```时, 可以直接拿到键和值。键和值是直接以数组的形式返回的，也就是说数组的第一个元素就是当前的键名，第二个元素就是值。这就可以配合数组的解构语法，直接拿到键和值。

```js
const m = new Map();
m.set('foo', '123');
m.set('bar', '345');

for (const item of m) {
    console.log(item); // ['foo', '123'];
}

for (const [key, value] of m) {
    console.log(key, value); // 'foo', '123'
}
```

```for...of```不能直接遍历普通对象的，他要求被遍历的对象必须存在一个叫做```Iterable```的接口。

```js
const obj = {
    store: [1, 2, 3, 4, 5],
    [Symbol.iterator]: function() {
        let index = 0;
        const self = this;
        return {
            next: function() {
                const result = {
                    value: self.store[index],
                    done: index >= self.store.length
                }
                index++;
                return result;
            }
        }
    }
}

for (const item of obj) {
    console.log(item); // 1, 2, 3, 4, 5
}
```

## 生成器

生成器函数generator是为了能够在复杂的异步编程中减少回调函数嵌套产生的问题，从而去提供更好的异步编程解决方案。

```js
function * foo() {
    return 100;
}
const result = foo();
console.log(result);
```

生成器函数在使用会配合```yield```的关键字，生成器函数会自动返回一个生成器对象，调用这个生成器的```next```会让这个函数的函数体开始执行，执行过程中一旦遇到```yield```关键词函数的执行就会被暂停下来，而且```yield```的值将会被作为```next```的结果返回，继续调用```next```函数就会从暂停的位置继续向下执行到下一个```yield```直到这个函数完全结束。

```js
const * foo() {
    console.log(1111);
    yield 100;
    console.log(2222);
    yield 200;
    console.log(3333);
    yield 300;
}
```

生成器函数最大的特点就是惰性执行，每调用一次```next```就会执行一次```yield```。比如实现一个发号器。


```js
function * createId() {
    let id = 1;
    while(true) {
        yield id++;
    }
}
const id = createId();

id.next().value;
```

## Array.prototype.include

检查数组中是否存在某个元素，可以判断数组当中是否存在某个指定的元素了，并且他返回的是一个布尔值，而且可以判断```NaN```。

## **指数运算符

```js
Math.pow(2, 10); // 表示2的10次方。
```

指数运算符是语言本身的运算符，就像加减乘除运算符一样，用起来非常简单。

```js
2 ** 10; // 2的10次方
```

## Object.keys/values/entries

```Object.keys```返回的是所有的键组成的数组，```Object.values```返回的是所有值组成的数组。

```Object.entries```将对象转成数组，每个元素是键值对的数组，可以快速将对象转为```Map```

```js
const l = Object.entries({a: 1, b: 2});
const m = new Map(l);
```

## Object.getOwnPropertyDescriptors

```Object.assign```复制时，将对象的属性和方法当做普通属性来复制，并不会复制完整的描述信息，比如```this```等.

```js
const p1 = {
    a: 'y',
    b: 'd',
    get name() {
        return `${this.a} ${this.b}`;
    }
}

const p2 = Object.assign({}, p1);
p2.a = 'z';
p2.name; // y d; 发现并没有修改到a的值，是因为this仍旧指向p1
```

使用```Object.getOwnPropertyDescriptors```获取完整描述信息

```js
const description = Object.getOwnPropertyDescriptors(p1);
const p2 = Object.defineProperties({}, description);
p2.a = 'z';
p2.name; // z d
```

## String.prototype.String/padStart

用给定的字符串在尾部拼接到指定长度

```js
'abc'.padEnd(5, '1');  // abc11;
```

用给定的字符串在首部拼接到指定长度

```js
'abc'.padStart(5, '1'); // 11abc;

```

## 允许对象和数组在最后添加一个逗号

```js
[1, 2, 3, ]

{a: 1, b: 2, }
```

## 33. async await

在函数声明时加入```async```关键字，则函数会变为异步函数，当使用```await```调用时，只有等到被```await```的```promise```返回，函数才会向下执行。

```js
const as = async () => {
    const data = await ajax();
}
```

## 收集剩余属性

将对象或者数组的剩余属性收集到新对象中，Map、Set、String 同样支持该能力。

```js
const data = {a: 1, b: 2, c: 3, d: 4};
const {a, b, ...arg} = data;
console.log(arg); // {c: 3, d: 4};
```

## for of 异步迭代

之前想要实现异步迭代要在```for of```外层嵌套一个```async```函数

```js
async function () {
    for (const fn of actions) {
        await fn();
    }
}
```

```ES2018```提供了一种新的书写方式。

```js
async function() {
    for await (const fn of actions) {
        fn();
    }
}
```

## JSON 成为 ECMAScript 的完全子集

```ECMAScript```优化行分隔符```\u2028```和段分隔符```\u2029```导致```JSON.parse```抛出语法错误异常问题。

```JSON.stringify```也做了改进，对于超出```Unicode```范围的转义序列，```JSON.stringify()```会输出未知字符：

```js
JSON.stringify('\uDEAD'); // '"�"'
```

## 修正Function.prototpye.toString

在以前，返回的内容中```function```关键字和函数名之间的注释，以及函数名和参数列表左括号之间的空格，是不会被显示出来的。 现在会精确返回这些内容，函数如何定义的，这就会如何显示。

## Array.prorptype.flat/flatMap

对数组进行降维，可以接收一个参数，用于指定降多少维，默认为```1```，降维最多降到一维。

```js
const array = [1, [2, [3]]]
array.flat() // [1, 2, [3]]
array.flat(1) // [1, 2, [3]]，默认降 1 维
array.flat(2) // [1, 2, 3]
array.flat(3) // [1, 2, 3]，最多降到一维
```

```flatMap()```允许在对数组进行降维之前，先进行一轮映射，用法和```map()```一样。然后再将映射的结果降低一个维度。可以说```arr.flatMap(fn)```等效于```arr.map(fn).flat(1)```。但是根据```MDN```的说法，```flatMap()```在效率上略胜一筹。

```flatMap()```也可以等效为```reduce()```和```concat()```的组合，下面这个案例来自```MDN```。

```js
var arr1 = [1, 2, 3, 4];
 
arr1.flatMap(x => [x * 2]);
// 等价于
arr1.reduce((acc, x) => acc.concat([x * 2]), []);
// [2, 4, 6, 8]
```

## String.prototype.trimStart/trimEnd

用过字符串```trim()```的都知道这两个函数各自负责只去掉单边的多余空格。

## Object.fromEntries()

```Object.entries()```的逆过程，可以将数组转化为对象。

## description of Symbol

```Symbol```是新的原始类型，通常在创建```Symbol```时会附加一段描述。只有把这个```Symbol```转成```String```才能看到这段描述，```ES2019```为```Symbol```新增了```description```属性，专门用于查看这段描述。

```js
const sym = Symbol('The description');
String(sym) // 'Symbol(The description)'
sym.description // 'The description'
```

## try...catch

过去，```catch```后面必须有一组括号，里面用一个变量代表错误信息对象。现在这部分是可选的了，如果异常处理部分不需要错误信息，可以把它省略。

```js
try {
  throw new Error('Some Error')
} catch {
  handleError() // 这里没有用到错误信息，可以省略 catch 后面的 (e)。
}
```

## Array.prototype.sort

```JavaScript```中内置的数组排序算法使用的是不稳定的排序算法，也就是说在每一次执行后，对于相同数据来说，它们的相对位置是不一致的。

```js
var arr1 = [{a: 1, b: 2}, {a: 2, b: 2}, {a: 1, b: 3}, {a: 2, b: 4}, {a: 5, b: 3}];
arr1.sort((a, b) => a.a - b.a);
```

返回的结果第一次可能是这样的:```[{a: 1, b: 2}, {a: 1, b: 3}...]```

但是第二次就变成:```[{a:1,b:3}, {a:1, b: 2}....]```

```JavaScript```放弃了不稳定的快排算法，而选择使用```Tim Sort```这种稳定的排序算法。优化了这个功能。

## 类的私有属性

在变量或函数前面添加一个```#```可以将属性保留为类内部使用。

```js
class Message {
 #message = "Howdy"
 greet() { console.log(this.#message) }
}

const greeting = new Message()

greeting.greet() // Howdy 内部可以访问
console.log(greeting.#message) // Private name #message is not defined  不能直接被访问
```
## Promise.allSettled

```Promise.allSettled```可以创建一个新的```Promise```，只在所有传递给它的```Promise```都完成时返回一个数组，其中包含每个```Promise```的数据。

```js
const p1 = new Promise((res, rej) => setTimeout(res, 1000));

const p2 = new Promise((res, rej) => setTimeout(rej, 1000));

Promise.allSettled([p1, p2]).then(data => console.log(data));

// [
//   Object { status: "fulfilled", value: undefined},
//   Object { status: "rejected", reason: undefined}
// ]
```

## 合并空运算符 ??

假设变量不存在，希望给系统一个默认值，一般会使用```||```运算符。但是在```javascript```中```空字符串```，```0```，```false```都会执行```||```运算符，```ECMAScript2020```引入合并空运算符解决该问题，只允许在值为```null```或```undefined```时使用默认值。

```js
const name = '';

console.log(name || 'yd'); // yd;
console.log(name ?? 'yd'); // '';
```

## 可选链运算符 ?.

业务代码中经常会遇到这样的情况，```a```对象有个属性```b```,```b```也是一个对象有个属性```c```。

```js
const a = {
    b: {
        c: 123,
    }
}
```

访问```c```，经常会写成```a.b.c```，但是如果```b```不存在时，就会出错。

```ECMAScript2020```定义可选链运算符解决该问题，在```.```之前添加一个```?```将键名变成可选。

```js
let person = {};
console.log(person?.profile?.age ?? 18); // 18
```

## bigInt

```JavaScript```可以处理的最大数字是```2的53次方 - 1```，可以在可以在```Number.MAX_SAFE_INTEGER```中看到。

```ECMAScript2020```引入```BigInt```，通过把字母```n```放在末尾, 可以运算大数据。它可以由数字和十六进制或二进制字符串构造。此外它还支持```AND```、```OR```、```NOT```和```XOR```之类的按位运算。唯一无效的位运算是零填充右移运算符。

```js
const bigNum = 100000000000000000000000000000n;
console.log(bigNum * 2n); // 200000000000000000000000000000n

const bigInt = BigInt(1);
console.log(bigInt); // 1n;

const bigInt2 = BigInt('2222222222222222222');
console.log(bigInt2); // 2222222222222222222n;
```

## 动态导入import

```import('./a.js')```返回一个Promise对象。

```js

const a = 123;
export { a };

```

```js
import('./a.js').then(data => {
    console.log(data.a); // 123;
})
```

## globalThis

标准化方式访问全局对象，在浏览器中```window```作为全局对象，在```node```中```global```作为全局对象，```ECMAScript2020```提供```globalThis```作为语言的全局对象，方便代码移植到不同环境中运行。

## String.prototype.replaceAll

```js
'abc111'.replaceAll('1', '2'); // abc222
```

## Promise.any

只要有一个```promise```是```fulfilled```时，则返回一个```resolved promise```，所有的```promise```都是```rejected```时，则返回一个```rejected promise```

```js
Promise.any([ Promise.reject(1), Promise.resolve(2) ])
 .then(result => console.log('result:', result))
 .catch(error => console.error('error:', error)); // result: 2 
```

## 逻辑赋值运算符

逻辑赋值运算符由逻辑运算符和赋值表达式组合而成：

```js
a ||= b;
// 与 a ||= b 等价
a || (a = b);
// 与 a ||= b 等价
if (!a) {
    a = b;
}


a &&= b;
// 与 a &&= b 等价
a && (a = b);
// 与 a &&= b 等价
if (a) {
    a = b;
}

a ??= b;
// 与 a ??= b 等价
a ?? (a = b);
// 与 a ??= b 等价
if (a === null || a === undefined) {
    a = b;
}
```

注意

```js
a = a || b; // 与 a ||= b 不等价
a = a && b; // 与 a &&= b 不等价
a = a ?? b; // 与 a ??= b 不等价
```

## 数字分隔符

使用```_```对数字进行分割，提高数字的可读性。

```js
// 1000000000 不易辨识
const count1 = 1000000000;

// 1_000_000_000 很直观
const count2 = 1_000_000_000;

console.log(count1 === count2); // true
```

## WeakRefs

```WeakRef```实例可以作为对象的弱引用，对象的弱引用是指当该对象应该被```GC```回收时不会阻止```GC```的回收行为。而与此相反的，一个普通的引用（默认是强引用）会将与之对应的对象保存在内存中。只有当该对象没有任何的强引用时，```JavaScript```引擎```GC```才会销毁该对象并且回收该对象所占的内存空间。因此，访问弱引用指向的对象时，很有可能会出现该对象已经被回收。

```js
const ref = new WeakRef({ name: 'koofe' });
let obj = ref.deref();
if (obj) {
  console.log(obj.name); // koofe
}
```

对于```WeakRef```对象的使用要慎重考虑，能不使用就尽量不要使用。
