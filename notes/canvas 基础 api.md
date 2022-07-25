## 1. 概述

```canvas```只能通过```width```和```height```设置宽高，不能通过```style```样式设置，```style```中设置的是拉伸效果。同时```canvas```是通过路径操作```moveto```、```lineto```、```stroke```进行绘制，一切路径操作之前，都需要```beginPath();```。

```js
const oC = document.getElementById('c1');
const ctx = oC.getContext('2d');
ctx.moveTo(100, 100);
ctx.lineTo(200, 200);
ctx.strokeStyle = 'red';
ctx.stroke();
// ctx.fill();

ctx.beginPath(); // 重新开始一条路径。
ctx.moveTo(300, 100);
ctx.lineTo(400, 200);
ctx.strokeStyle = 'yellow';
ctx.stroke();
```

```beginPath```表示清除之前的路径，必须添加。```closePath```是闭合当前的路径，在需要的时候选择加，如果不使用```closePath```图形不会真的闭合。

图形不能修改只能删了重画同时绘制的图形也没有事件而且绘制速度极快。


## 2. 绘制直线

```js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
ctx.beginPath();
// 设置粗细
ctx.lineWidth = 3;
// 设置线条颜色
ctx.strokeStyle = 'red';
// 起点 终点 中间点
ctx.moveTo(100, 100);
ctx.lineTo(300, 300);
ctx.lineTo(300, 400);
ctx.stroke();
ctx.closePath();
```

## 3. 高清绘制

```window.devicePixelRatio```像素比。表示设置的像素和实际像素的比例，如果值为``2``表示实际像素是我们设置的```2倍```，如果设置```100```宽，实际就是```200```宽，但我们编写页面用了```10```像素会被放置在```20```像素的容器中，页面就会显得模糊。

可以将画布按照像素比调整为和像素一致，再用```style```调整回最初的大小。

```js

const getPixelRatio = (context) => {
    return window.devicePixelRatio || 1;
}

const ratio = getPixelRatio();
const oldWidth = canvas.width;
const oldHeight = canvas.height;

canvas.width = canvas.width * ratio;
canvas.height = canvas.height + ratio;

canvas.style.width = oldWidth + 'px';
canvas.style.height = oldHeight + 'px';

ctx.scale(ratio, ratio);
```

## 4. 坐标系绘制

![屏幕快照 2021-06-26 20.54.14.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13b0613d7a7c47858ddc37d77e52ba50~tplv-k3u1fbpfcp-watermark.image)

```js
// 提前设置相关属性
const ht = canvas.clientHeight
const wd = canvas.clientWidth
const pad = 20
const bottomPad = 20
const step = 100

const drawAxis = (options) => {
const { ht, wd, pad, bottomPad, step, ctx } = options
// 绘制坐标轴
ctx.beginPath()
ctx.lineWidth = 2
ctx.strokeStyle = 'lightblue'
ctx.moveTo(pad, pad)
ctx.lineTo(pad, ht * 1.5 - bottomPad)
ctx.lineTo(wd * 1.5 - pad, ht * 1.5 - bottomPad)
ctx.stroke()
ctx.closePath()

// 绘制 X 轴方向刻度
ctx.beginPath()
ctx.lineWidth = 1
ctx.strokeStyle = '#666'
for (let i = 1; i < Math.floor(wd * 1.5 / step); i++) {
    ctx.moveTo(pad + i * step, ht * 1.5 - bottomPad)
    ctx.lineTo(pad + i * step, ht * 1.5 - bottomPad - 10)
}
ctx.stroke()
ctx.closePath()


// 绘制 Y 轴方向刻度
ctx.beginPath()
ctx.lineWidth = 1
ctx.strokeStyle = '#666'
for (let i = 1; i < Math.floor(ht * 1.5 / step); i++) {
    ctx.moveTo(pad, (ht * 1.5 - bottomPad) - (i * step))
    ctx.lineTo(pad + 10, (ht * 1.5 - bottomPad) - (i * step))
}
ctx.stroke()
ctx.closePath()
}

drawAxis({
    ht: ht,
    wd: wd,
    pad: pad,
    bottomPad: bottomPad,
    step: step,
    ctx: ctx
})
```

## 5. 绘制矩形

```js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.lineWidth = 5;
// 边框颜色
ctx.strokeStyle = 'red';
// 填充颜色
ctx.fillStyle = 'green';
// 起点坐标，宽，高
ctx.rect(100, 100, 300, 200);
// 填充
ctx.fill();
// 描边
ctx.stroke();
ctx.closePath();
```

## 6. 直方图绘制

![屏幕快照 2021-06-26 20.57.30.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b424f81588f466c9d81346914912e74~tplv-k3u1fbpfcp-watermark.image)

```js
// 绘制坐标轴
ctx.beginPath()
ctx.lineWidth = 2
ctx.strokeStyle = 'lightblue'
ctx.moveTo(pad, pad)
ctx.lineTo(pad, ht * 1.5 - bottomPad)
ctx.lineTo(wd * 1.5 - pad, ht * 1.5 - bottomPad)
ctx.stroke()
ctx.closePath()

// 绘制 X 轴方向刻度
ctx.beginPath()
ctx.lineWidth = 1
ctx.strokeStyle = '#666'
for (let i = 1; i < Math.floor(wd * 1.5 / step); i++) {
    ctx.moveTo(pad + i * step, ht * 1.5 - bottomPad)
    ctx.lineTo(pad + i * step, ht * 1.5 - bottomPad + 10)
}
ctx.stroke()
ctx.closePath()


// 绘制 Y 轴方向刻度
ctx.beginPath()
ctx.lineWidth = 1
ctx.strokeStyle = '#666'
for (let i = 1; i < Math.floor(ht * 1.5 / step); i++) {
    ctx.moveTo(pad, (ht * 1.5 - bottomPad) - (i * step))
    ctx.lineTo(pad + 10, (ht * 1.5 - bottomPad) - (i * step))
}
ctx.stroke()
ctx.closePath()
}

drawAxis({
    ht: ht,
    wd: wd,
    pad: pad,
    bottomPad: bottomPad,
    step: step,
    ctx: ctx
})

// 绘制直方图
ctx.beginPath()
for (var i = 1; i < Math.floor(wd * 1.5 / step); i++) {
    const height = Math.random() * 300 + 50
    ctx.fillStyle = '#' + parseInt(Math.random() * 0xFFFFFF).toString(16)
    ctx.fillRect((i * step), ht * 1.5 - bottomPad - height, 40, height)
}
ctx.closePath()
```

## 7. 绘制圆形

```js
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
ctx.beginPath();
ctx.lineWidth = 5;
// 边框颜色
ctx.strokeStyle = 'red';
// 填充颜色
ctx.fillStyle = 'green';
// 起点坐标，半径，起点弧度，终点弧度, 顺时针还是逆时针
ctx.arc(100, 100, 100, 0, Math.PI * 2, false);
// 填充
ctx.fill();
// 描边
ctx.stroke();
ctx.closePath();
```

## 8. 饼图绘制

![屏幕快照 2021-06-26 20.57.54.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/567ee6cb630445029b7282c6a5bafb3d~tplv-k3u1fbpfcp-watermark.image)

```js
// 提前设置相关属性
const ht = canvas.clientHeight
const wd = canvas.clientWidth
const pad = 20
const bottomPad = 20
const step = 100

const drawAxis = (options) => {
const { ht, wd, pad, bottomPad, step, ctx } = options
// 绘制坐标轴
ctx.beginPath()
ctx.lineWidth = 2
ctx.strokeStyle = 'lightblue'
ctx.moveTo(pad, pad)
ctx.lineTo(pad, ht * 1.5 - bottomPad)
ctx.lineTo(wd * 1.5 - pad, ht * 1.5 - bottomPad)
ctx.stroke()
ctx.closePath()

// 绘制 X 轴方向刻度
ctx.beginPath()
ctx.lineWidth = 1
ctx.strokeStyle = '#666'
for (let i = 1; i < Math.floor(wd * 1.5 / step); i++) {
    ctx.moveTo(pad + i * step, ht * 1.5 - bottomPad)
    ctx.lineTo(pad + i * step, ht * 1.5 - bottomPad + 10)
}
ctx.stroke()
ctx.closePath()

// 绘制 Y 轴方向刻度
ctx.beginPath()
ctx.lineWidth = 1
ctx.strokeStyle = '#666'
for (let i = 1; i < Math.floor(ht * 1.5 / step); i++) {
    ctx.moveTo(pad, (ht * 1.5 - bottomPad) - (i * step))
    ctx.lineTo(pad + 10, (ht * 1.5 - bottomPad) - 