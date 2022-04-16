
## 1. 重置css

```Reset.css```重置初始样式。

```Normalize.css```修复不规则样式。
```Neat.css```融合了```Reset```和```Normalize```。

如果开发```H5```页面，最好全局添加如下样式。

```css
html { box-sizing: border-box;}
*,*:before, x:after { box-sizing: inherit;}
```

## 2. no-image

通过```css```实现的```icon```。

```s
http://cssicon.space
```

## 3. css hint

不要使用多个```class```选择元素，如```a.foo.boo```，移除空的```css```，正确使用属性。

使用```csshint```。

```s
npm install csshint

# csslint.net
```

## 4. BFC IFC GFC FFC

```Box```是```css```的布局基本单位，直观点来说就是一个页面是由很多个```Box```组成，元素的类型和```display```属性决定了这个```Box```的类型，不同类型的```Box```会参与不同的```Formatting Context```也就是一个决定如何渲染文档的容器。因此```Box```内的元素会以不同的方式渲染。

```block-level box```：```display```属性为```block```、```inline-block```、```inline-table```的元素，会产生```inline-leve box```，并且参与```inline formatting content```。

```inline-level box```：```display```属性为```inline```、```inline-item```、```table```的元素会生成```block-level box```并参与```block formatting Context```。

```Fromatting context```是```w3c css2.1```规范中的一个概念，他是页面中的一款渲染区域，并且有一套渲染规则，他决定了其子元素将如何定位，以及和其他元素的关系和相互作用。最常见的```Formatting Context```有```block fomatting context```简称```BFC```和```inline formatting context```简称```IFC```。

会生成BFC的元素：根元素```float```不为```none```， 绝对定位或```fixed```定位，```overflow```不为```visiable```，块级```css```属性。

```FFC```是```flex```布局的渲染。
```GFC```是```Grid Layout```。

## 5. 数学矩阵技巧

```SVG```

```Canvas```

```webgl```

```css3d```

```s
http://wow.techbrood.com/fiddle/25741
```

```css-matrix3d```

注意事项

1. 不要直接定义子节点，应把共性声明放到父类

2. 结构和皮肤相分离

3. 容器和内容相分离

4. 抽象出可重用的元素，建好组件库，在组件库内寻找可用的元素组装页面

5. 往你想要扩展的对象本身增加class而不是他的父节点

6. 对象应保持独立性

7. 避免使用Id选择器，权重太高，无法重用

8. 避免位置相关的样式。

9. 保证选择器相同的权重

10. 类名 简短 清晰 语义化

```s
http://oocss.org

# 下载 Alternate download
```

## 6. css 后处理器

```autoprefixer```、```postcss```。

## 7. css分层

```css```有语义化的命名约定和```css```层的分离，将有助于它的可扩展性，性能的提高和代码的组织管理大量的样式，覆盖，权重和很多```!important```。分好层可以让团队命名统一规范，方便维护。有责任感的去命名选择器。

### 1. SMACSS

```SMACSS```意思是分文件```base.css```、```layout.css```、```module.css```、```state.css```。

```base```：设定标签越苏的预设值。

```layout```：整个网站的大架构的外观。

```module```：应用在不同页面的公共模块。

```state```：定义元素不同的状态。

```theme```：画面上所有主视觉的定义，```border-color```、```background```。

修饰符用```-```,子元素用```__```。

### 2. BEM

```BEM```和```SMACSS```非常类似，主要用来如何给项目命名，一个简单命名更容易让别人一起工作，比如选项卡导航是一个块（```Block```），这个块里的元素的是其中标签之一，而当前选项卡是一个修饰状态。

```block```代表了更高级别的抽象或组件。

```block__element```代表```block```的后台，用于形成一个完整的```block```的整体。

```.block--modifier```代表```block```的不同状态。

修饰符使用的是```_```，子模块使用```__```符号。（不用一个```-```的原因是```css```单词连接）。

### 3. SUIT

### 4. ACSS

原子化的```css```，小粒度的```css```。

考虑如何设计一个系统的接口，原子是框架一个区块的最基本的特质，比如说表单，由于多个表单组件组成，网站由多个模块组成。

```css
.fl { float: left}
```

### 5. ITCSS

```ITCSS```：混合```css```，```minx```。

## 8. nextcss

```s
https://cssnext.github.io/playground
```

```css
:root { // 全局变量
    --fontSize: 1rem; // 变量
    --mainColor: #12345678;
    --highlightColor: hwb(190, 35%, 20%)
}

body {
    color: var(--mainColor)
}

:root {
    --centered: {
        display: flex;
        align-items: center;
        justify-content: center;
    }
}
.centered {
    @apply --centered;
}
```

```calc```代表```css```表达式。

```css
calc(15px * 2);

image-set (
    url(img/test.png) 1x,
    url(img/test-2x.png) 2x
)
```

```filter```滤镜



```css
// css正则
[frame=hsides i] { // 忽略hsides大小写

}
```

使用```webpack```的```css-loader```编译添加```modules```。

使用```postcss```进行编译```css```。

使用```postcss-preset-env```编译```cssnext```语法。

postcss插件集

```postcss-custom-properties```运行时变量。

```postcss-simple-vars```与```scss```一致的变量实现。

```postcss-mixins```实现类似```sass```的```@mixin```的功能。

```postcss-extend```实现类似```sass```的继承功能。

```postcss-import```实现类似```sass```的```import```。

```cssnext```面向未来，```cssgrace```修复过去，兼容```ie6```。

```s
https://cssdb.org
```

```autoprefixer```已经被继承，不需要用了。

### css-doodle

是一个```webcomonent```，```https://css-doodle.com/```。

```html
<css-doodle>
</css-doodle>
```

## 9. Houdini

在现今的```web```开发中，```js```几乎占据所有版面，除了控制页面逻辑与操作```DOM```对象外，连```css```都直接写在````js````里面了，就算浏览器还没实现的特性，总会有人做出对应的```polyfills```，让你快速的将新```Feature```应用到```Production```环境中，更别提````Babel````等工具帮忙转移。

而```css```就不同了，除了制定了```css```标准规范所需的时间外，各家浏览器的版本，实战进度差异更是旷日持久，顶多利用```postcss```，```sass```等工具来帮转译出浏览器能接受的```css```，开发者们能操作的就是通过```js```去控制```DOM```与```CSSOM```来影响页面的变化，但是对于接下来的```layout```，```paint```与```composite```就几乎没有控制权了。

为了解决上述问题，为了让css的魔力不再被浏览器把持，```houdini```就诞生了。```css houdini```让开发者能够介入浏览器的```css engine```。

允许开发者自由扩展```css```此法分析器```Parser```，```Paint```，```Layout```。

```worklets```的概念和```web worker```类似，允许引入脚本文件并执行特定的```js```代码，这样的```js```代码要满足两个条件，第一可以在渲染流程中调用，第二和主线程独立。

```worklet```脚本严格控制了开发者所能执行的操作类型，这就保证了性能，```worklets```的特点就是轻量以及生命周期较短。

```js
CSS.paintWorklet.addModule('xxx.js');
CSS.layoutWorklet.addModule('xxx.js');
// xxx.js
registerPaint('xxx', class {
    static get inputProperties() {}
    static get inputArguments() {}
    paint(ctx, geom, props) {}
})

```

演示```css```。

```css
.el {
    --elUnit: 500px;
    --arcColor: yellow;
    height: var(--elUnit);
    width: var(--elUnit);
    --background-canvas: (ctx, geom) > { // 变量，可以使用函数
        // geom 当前类 .el 所有的信息
        // ctx相当于一个canvas
        ctx.strokeStyle = `var(--arcColor)`;
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.arc(200, 200, 50, 0, 2*Math.PI);
        ctx.stroke();
        ctx.closePath();
    };
    background: paint(--background-canvas);
}
```

需要在```js```中进行激活。

```html
<script>
CSS.paintWorklet.addModule('./arc.js');
</script>
```

```arc.js```

```js
if (typeof registerPaint !== 'undefined') {
    registerPaint('background-canvas', class {
        static get inputProperties() { // 获取变量
            return ['--background-canvas'];
        }
        paint(ctx, geom,  properties) { // 绘制
            eval(properties.get('--background-canvas').toString())(
                ctx, geom, properties
            );
        }
    })
}
```

资源文件需要放入服务器中，起个服务进行加载。```js```类似```canvas```语法，只是不能操作```dom```。

```js
class Test {
    static get inputProperties() { // 获取变量
        // const a = 123;
        // setTimeout()
        return ['--background-canvas'];
    }
    paint(ctx, geom,  properties) { // 绘制
        eval(properties.get('--background-canvas').toString())(
            ctx, geom, properties
        );
    }
}

registerPaint('background-canvas', Test);
```

```js```中的写法。

```js
CSS.px(42);  === 42px;

let pos = new CSSPositionValue(
    new CSSUnitValue(5, 'px'),
    new CSSUnitValue(4, 'px')
);
```

数字值总是以数字形式返回，而不是字符串。

```js
el.style.opacity += 0.1, el.style === '0.30.1' // dragons!
```

算数运算和单位转换，在绝对长度单位，例如```px-cm```之间进行转换并进行进本的数学运算。

数值阀内限制和舍入，```typed OM```通过对值进行范围限制和舍入，以使其在属性的可接受范围内。

更好的性能，浏览器必须做更少的工作序列化和反序列化字符串值，现在对于```CSS```值，引擎可以对```JS```和```C++```使用相似的理解，```Tab Akis```已经展示了一些早起的性能基准测试，与使用旧的```CSSOM```和字符串相比，```typed om```的运行速度快了大约```30%```，这对使用```requestionAnimationFrame```处理快速```css```动画可能很重要。

```crbug.com/808933```可以跟踪```Blink```的更多性能演示。

错误处理，新的解析方法带来了```css```世界中的错误处理。

不需要猜测名字是驼峰还是字符串，```typed OM```中的```css```属性名称始终是字符串，与实际在```css```中编写的内容一致。

1. 居中

父元素```display:flex```。
子元素```margin: auto```。

```css
.parent {
    display: flex;
}
.children {
    margin: auto;
}
```

2. 居右
父元素```display: flex```。
子元素```margin-left: auto```。

```css
.parent {
    display: flex;
}
.children {
    margin-left: auto;
}
```

3. 色相

```css
filter:hue-rotate(60deg); // 色相旋转60度
```

4. 蒙版

```css
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
```

5. 设置滚动条

```css
div::-webkit-scrollbar {
    width: 200px;
}
```

6. 拖动

```css
resize: horizontal;
```
7. 修改滚动条

```css
-webkit-scrollbar: // 滚动条
-webkit-scrollbar-button:  // 滚动条上下箭头
-webkit-scrollbar-thumb: // 滚动按钮
-webkit-scrollbar-track: // 滚动区域
-webkit-scrollbar-track-piece: // 剩余滚动区域
-webkit-scrollbar-comer: // 横纵滚动交界
-webkit-scrollbar-resizer: // 纵向滚动交界
```

8. 遮罩

```css
-webkit-mask-image: url(png文件遮罩)
```

9. 混合模式

```css
mix-blend-mode: hue; // 屏蔽灰色之后着色。
background-blend-mode: darken; // 背景混合模式
```

10. 滤镜

```css
filter: brightness(80%) grayscale(20%) contrast(1.2); // 光线下降80%；灰度20%；对比度
```

白天变黑夜

```css
background-blend-mode: darken;
filter: brightness(80%) grayscale(20%) contrast(1.2);
```

11. 自助滚动

父元素

```css
white-space: nowrap;
scroll-snap-align: x mandatory;
```

子元素固定宽度。

```css
scroll-snap-align: start;
```

12. 画图形

```css
shape-outsize: polygon(0, 0, 25%, 0, 57%);
```

响应式布局核心知识

```css
repeat() // 函数
minimax() // 函数
auto-fill/auto-fit
grid-auto-flow
max-content/min-content
```

13. 阴影

文本

```css
text-shadow: 横向偏移 纵向偏移 大小 颜色
```

盒子

```css
box-shadow: [insert] 横向偏移 纵向偏移 大小 [阴影区] 颜色
```
14. 渐变，一般只能加在背景上

线性渐变

```css
background-image: linear-gradient(left top, green);
```

径向渐变

```css
background-image: radial-gradient(left top, green);
```

15. animation

```css
animation-name: // 动画名称
animation-duration: // 时长
animation-timing-function: // ease 运动方式
animation-fill-mode: forwards; // 结束填充模式，恢复还是保留
animation-iteration-count: // 循环次数
animation-direction: alternate; // 循环方向
animation-play-state: paused; // 暂停
```

## 10. 3D

```s
720yun.com 
h5doo.com
```

## 11. 陀螺仪

陀螺仪又叫角速度传感器。是不同于加速计(```G-sensor```)的，他的测量物理量是偏移，倾斜时的转动角速度，在手机上，仅用加速度计没办法测量或重构出完整的```3D```动作，也就是测不到转动的动作的，```G-sensor```只能检测轴向的线性动作。但陀螺仪则可以对转动、偏移的动作做很好的测量，这样就可以精确分析判断出使用者的实际动作。而后根据动作，可以对手机做相应的操作。

沿着```Y```轴转动，是```gamma```角。

沿着```Z```轴转动，是```alpha```角。

沿着```X```轴转动，是```beta```角。


```deviceorientation```设备的物理方向信息，表示为一系列本地坐标系的旋转角。

```devicemotion```提供设备的加速信息，```g```的加速度，空气阻力之类都可以算出。

```compassneedscalibration```用于通知```Web```站点使用罗盘信息校准上述事件。

获取角度

```js
window.addEventListener('deviceorientation', (event) => {
    // event.alpha, 0, -360
    // event.beta -180, 180
    // event.gamma, -90, 90
}, true)
```

罗盘校准

```js
window.addEventListener('compassneedscalibration', (event) => {
    alert('罗盘需要校准');
    event.preventDefault();
}, true)
```

获取重力加速度，运动状态下的角度

```js
window.addEventListener('devicemotion', (event) => {
    // 处理event.acceleration
    // x(y, z): 设备在 x(y, z) 方向上的移动加速度值
    // event.accelerationIncludingGravity
    // 考虑了重力加速度后设备在x(y, z)
    // event.rotationRate
    // alpha, beta, gamma: 设备绕x,y,z轴旋转的角度
}, true)
```


重力加速度(```Gravitational acceleration```) 是一个物体受重力作用的情况下所具有的加速度。
与位置有关(```G = mg```) (其中```g=9.80665m/s^2```为标准重力加速度)。

摇一摇代码：

```js
var speed = 30;
var x = y = z = lastX = lastY = lastZ = 0;
function deviceMotionHandler(eventData) {
    var acceleration = event.accelerationIncludingGravity;
    x = acceleration.x;
    y = acceleration.y;
    z = acceleration.z;
    if (Math.abs( x - lastX) > speed || Math.abs(y - lastY) > speed || Math.abs(z - lastZ) > speed) {
        alert(1);
    }
}
```

```css3d-engine```库

```js
var s = new C3D.Stage();
s.size(window.innerWidth, window.innerHeight).material({
    color: '#ccc',
}).update();
document.getElementById('main').appendChild(s.el);
// 创建1个立方体放入场景
var c = new C3D.Skybox();
c.size(1024).position(0, 0, 0).material({
    front: { image: 'images/cube_FR.jpg'},
    back: { image: 'images/cube_FR.jpg'},
    left: { image: 'images/cube_FR.jpg'},
    right: { image: 'images/cube_FR.jpg'},
    up: { image: 'images/cube_FR.jpg'},
    down: { image: 'images/cube_FR.jpg'},
}).update();
s.addChild(c);
```

```parallax.js```视差，轻量级引擎库。
