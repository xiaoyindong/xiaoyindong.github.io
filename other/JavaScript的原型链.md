## 1. 前言  

在```ES6```以前```javascript```并没有类的概念，也可以说他并没与创建类的```class```关键字，如果想用借助面向对象的思想来完成开发它就要自己实现类的写法。

```javascript```是一门基于原型的设计语言，这在语言设计之初就已经确定了，即便后来```ES6```中加入了```class```关键字，```extends```关键字，仍旧无法改变他作为原型继承的本质，```ES6```中```class```的实现方式更像是之前开发者通过```ES5```实现继承的语法糖，只是将大家习惯的写法合法化，规范化了。

## 2. 什么是原型

原型可以比作模型，或者说模具。比如说一个水杯，是圆的还是方的都是由模具决定的，如果模具的样子变了，那么生产出来的水杯也就变了。这样来描述，你可能会觉得差劲，叫什么原型啊，叫构造器不是更贴切。没错，在这个例子中，模具就是水杯的构造器(```constructor```)。

在这个过程中原型起到什么作用呢？同样的例子假设水杯是有额外功能的，比如加热或者制冷。那么问题就来了，构造器只能决定水杯的样子，是没办法给他提供加热和制冷的功能的。这个时候就需要原型(```prototype```)了。

一般情况，构造器决定样子，名称等固定的属性。原型决定的是功能，可以进行操作的方法。构造器(```constructor```)和原型(```prototype```)共同决定了一个物体的存在形式。同样对于用户来说真正使用的是水杯并不是构造器和原型，那也就是说构造器和原型是不能直接使用的。

构造器(```constructor```)和原型(```prototype```)的关系怎么来描述呢？原型的```constructor``` 是构造器，构造器的 ```prototype``` 是原型。

![image.png](https://upload-images.jianshu.io/upload_images/14119996-c93b4947682f7af7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在```javascript```中有一句话叫```万物皆对象```，每个对象都有原型。创建函数时可以采用```new```的方式调用，这种调用方式有个名字叫```实例化```。

```js
// 创建一个函数
function B(name) {
    this.name = name;
};
// 实例化
var bb = new B('实例化的b');
console.log(bb.name); // 实例化的b;
```

如上面的代码，```bb```是通过```B```实例化之后得到的对象。在这里```B```就是一个构造器，他所拥有的名字(```this.name```)属性会带给```bb```，这也符合之前杯子的例子，杯子的属性会从构造器中获得。

假如想要```bb```具有一定的功能，那么就需要在原型上下功夫了。根据上面构造器和原型的关系。可以这样做。

```js
// 创建一个函数

function B(name) {
    this.name = name;
};

// 在原型上添加一个方法
B.prototype.tan = function() {
    alert('弹出框');
}

// 实例化

var bb = new B('实例化的b');

console.log(bb.name); // 实例化的b;

bb.tan(); // alert('弹出框');

```

上面的代码中，在B的原型上添加了一个```tan```的方法，那么实例化出来的```bb```也具备了这个方法。

这里就简单实现了一个类。用下面一张图，说明一下。实例对象(```bb```), 原型(```prototype```), 构造函数(```constructor```)的关系。

![image.png](https://upload-images.jianshu.io/upload_images/14119996-c61e97c5dfc81be8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```B```是构造的一个类，称为构造函数。他用```prototype```指向了自己的原型。而他的原型也通过```constructor```指向了它本身。

```js
B.prototype.constructor === B;  // true;
```

```bb```和```B```没有直接的关联，虽然```B```是```bb```的构造函数，这里用虚线表示。```bb```有一个```__ proto__```属性，指向了```B```的```prototype```

```js
bb.__ proto__ === B.prototype; // true;
bb.__ proto__.constructor = B; // true;
```

总之

1，每创建一个函数```B```，就会为该函数创建一个```prototype```属性，这个属性指向函数的原型对象；

2，原型对象会默认去取得```constructor```属性，指向构造函数。

3，当调用构造函数创建一个新实例```bb```后，该实例的内部将包含一个指针```__ proto__```，指向构造函数的原型对象。

## 3. 默认原型

我们知道，所有引用对象都默认继承了```Object```，所有函数的默认原型都是```Object```的实例。
之前说过构造函数和原型之间具备对应关系，如下：

![image.png](https://upload-images.jianshu.io/upload_images/14119996-751a7f31555084ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

既然函数的默认原型都是```Object```的实例，```B```的原型对象也应该是```Object```的实例子，也就是说。```B```的原型的```__ proto__```应该指向```Objct```的原型。

![image.png](https://upload-images.jianshu.io/upload_images/14119996-bbe3c17a6217b5b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Object```的原型对象的原型是最底部了，所以不存在原型，指向```NULL```；

```js
console.log(Object.prototype.__ proto__); // null;
```

![image.png](https://upload-images.jianshu.io/upload_images/14119996-071378e3a37098af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. Function对象

我们知道，函数也是对象，任何函数都可以看作是由构造函数```Function```实例化的对象，所以```Function```与其原型对象之间也存在如下关系

![image.png](https://upload-images.jianshu.io/upload_images/14119996-661a7ae7754cc1c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果将```Foo```函数看作实例对象的话，其构造函数就是```Function()```，原型对象自然就是Function的原型对象；同样```Object```函数看作实例对象的话，其构造函数就是```Function()```，原型对象自然也是```Function```的原型对象。

![image.png](https://upload-images.jianshu.io/upload_images/14119996-750c436199405a44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果```Function```的原型对象看作实例对象的话，如前所述所有对象都可看作是```Object```的实例化对象，所以```Function```的原型对象的```__ proto __```指向```Object```的原型对象。

![image.png](https://upload-images.jianshu.io/upload_images/14119996-41333643103a6340.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这里```prototype```、```__ proto __```、```constructor```三者之间的关系我们就说完了。

## 5. 实现继承

```js
function Animal() {
    this.type = '动物';
}
Animate.prototype.eat = function() {
  console.log('吃食物');
}
```

上面定义了一个动物类，作为父类

```
function Cat(name) {
    this.name = name || ‘小猫’;
}
```

定义了一个猫作为子类，这里我们要继承动物类的```eat```方法和```type```属性

```js
function Cat(name){
  Animal.call(this);
  this.name = name || '小猫';
}
```

在实例化```Cat```时通过```call```执行了```Animal```类, 这样```Animal```中的```this```就被修改为当前```Cat```的```this```。所有的属性也会加在```Cat```上。

```js
(function(){
  // 创建一个没有实例方法的类
  var Super = function(){};
  Super.prototype = Animal.prototype;
  //将实例作为子类的原型
  Cat.prototype = new Super();
})();
```

通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化实例方法/属性。而父类的方法仍旧可以赋值给子类。

```Cat.prototype = new Super()```; 可以实现方法的继承，是因为，根据前面的知识我们知道 ```new Cat()```的```__ proto __```是指向 ```Cat```的原型的。

```js
(new Cat()).__ proto __ === Cat.prototype; // true
```

```new Cat()```所有的方法都是从原型上取到的。

我们通过 ```Cat.prototype = new Super()```; 公式变成了。

```(new Cat()).__ proto __ = Cat.prototype = new Super()```；

所以现在```(new Cat()).__proto__ ```指向了 ```Super```的```prototype```。也就是```new Cat```的方法是继承自```Super.prototype```。

```Super.prototype```又在前一句等于```Animal.prototype```。所以实现了```Cat```继承```Animal```。

这里我们就实现了```js```属性和方法的继承。不过还在最后一个小问题。

我们知道``` prototype``` 和 ````constructor```` 是相互指向的。

```Cat.prototype.constructor``` 应该等于 ```Cat```;

但是随着我们的修改了```Cat.prototype = Super.prototype```;

现在```Cat.prototype.constructor```是等于```Super```的。

所以我们还应该纠正这个问题，一句话搞定。

```js
Cat.prototype.constructor = Cat; // 需要修复下构造函数
```

以上就是```js```的原型继承，完整代码如下。

```js
// 创建一个父类
function Animal() {
    this.type = '动物';
}
// 给父类添加一个方法
Animate.prototype.eat = function() {
  console.log('吃食物');
}

// 创建一个子类
function Cat(name){
  // 继承Animal的属性
  Animal.call(this);
  this.name = name || '小猫';
}

// 继承 Animal 的方法

(function(){
  // 创建一个没有实例方法的类
  var Super = function(){};
  Super.prototype = Animal.prototype;
  //将实例作为子类的原型
  Cat.prototype = new Super();
})();
// 修正构造函数
Cat.prototype.constructor = Cat; 
```

好啦，```js```的继承原理和```prototype```,```__proto__```, ```constructor```之间的关系我们就说完了，```ES6```底层的实现方式原理基本相同。
