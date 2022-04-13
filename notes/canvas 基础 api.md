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

ctx.beginPath()
ctx.shadowOffsetX = 0
ctx.shadowOffsetY = 0
ctx.shadowBlur = 4
ctx.shadowColor = '#333'
ctx.fillStyle = '#5C1918'
ctx.moveTo(400, 300)
ctx.arc(400, 300, 100, -Math.PI / 2, -Math.PI / 4)
ctx.fill()
ctx.closePath()

ctx.beginPath()
ctx.shadowOffsetX = 0
ctx.shadowOffsetY = 0
ctx.shadowBlur = 4
ctx.shadowColor = '#5C1918'
ctx.fillStyle = '#A32D29'
ctx.moveTo(400, 300)
ctx.arc(400, 300, 110, -Math.PI / 4, Math.PI / 4)
ctx.fill()
ctx.closePath()

ctx.beginPath()
ctx.shadowOffsetX = 0
ctx.shadowOffsetY = 0
ctx.shadowBlur = 4
ctx.shadowColor = '#A32D29'
ctx.fillStyle = '#B9332E'
ctx.moveTo(400, 300)
ctx.arc(400, 300, 120, Math.PI / 4, Math.PI * 5 / 8)
ctx.fill()
ctx.closePath()

ctx.beginPath()
ctx.shadowOffsetX = 0
ctx.shadowOffsetY = 0
ctx.shadowBlur = 4
ctx.shadowColor = '#B9332E'
ctx.fillStyle = '#842320'
ctx.moveTo(400, 300)
ctx.arc(400, 300, 130, Math.PI * 5 / 8, Math.PI)
ctx.fill()
ctx.closePath()

ctx.beginPath()
ctx.shadowOffsetX = 0
ctx.shadowOffsetY = 0
ctx.shadowBlur = 4
ctx.shadowColor = '#842320'
ctx.fillStyle = '#D76662'
ctx.moveTo(400, 300)
ctx.arc(400, 300, 140, Math.PI, Math.PI * 3 / 2)
ctx.fill()
ctx.closePath()
```

## 9. 绘制文字

![屏幕快照 2021-06-27 14.24.31.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f73fb01baa044ad8b8afe638bd161269~tplv-k3u1fbpfcp-watermark.image)

```js
// 填充文字
ctx.fillStyle = 'origin';
ctx.font = 'bold 50px 微软雅黑'; // 字体大小
ctx.fillText('隐冬', 100, 100); // 坐标位置
// 描边文字
ctx.strokeStyle = 'red';
ctx.strokeText('隐冬', 100, 200);

// 对齐属性设置
// 水平方向
ctx.textAligin = 'center'; //left right start end center, left和start效果相同，right和end效果也相同。
// 垂直反向
ctx.textBaseline = 'middle'; // top bottom middle
ctx.fillText('隐冬', 400, 300);
```

## 10. 运动

```canvas```绘制运动的思路很简单，使用前面提到的```api```进行图案绘制，然后使用```clearRect```方法将绘制的内容清除，再使用```api```在不同的位置画出，这样就实现了动画。

这里使用```drawCircle```方法绘制一个圆，函数接收远点坐标的半径，然后使用```setInterval```定时器不停地调用这个方法就可以了。

```js

const drawCircle = (x, y, r) => {
    ctx.beginPath()
    ctx.fillStyle = 'orange'
    ctx.arc(x, y, r, 0, Math.PI * 2)
    ctx.fill()
    ctx.closePath()
}

// 配置属性
const wd = canvas.clientWidth * 1.5
const ht = canvas.clientHeight * 1.5
let x = y = 100
const r = 20
let xSpeed = 6
let ySpeed = 4

drawCircle(x, y, r)

setInterval(() => {
    ctx.clearRect(0, 0, wd, ht)  // 清空画布
    if (x - r <= 0 || x + r >= wd) {
        xSpeed = -xSpeed
    }

    if (y - r <= 0 || y + r >= ht) {
        ySpeed = -ySpeed
    }
    x += xSpeed
    y += ySpeed
    drawCircle(x, y, r)
}, 20)

```

![11111.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26bf478b3d8440b290a9329cad8164f3~tplv-k3u1fbpfcp-watermark.image)

可以同时定义多个小球。比如定义```20```个小球。还是很有意思的。一般```canvas```的动画就是这样来实现的。

![11111.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/209ef1d3e2b14833947b5d82c01a487c~tplv-k3u1fbpfcp-watermark.image)

## 11. 变换

图形操作要放在绘图之前，下笔之前考虑好。变换都是整个画布作为单位进行变换，可以将图形移动到画布左上角，中心位于画布左上角，转完再移动出来。
rotate是旋转，translate是平移，scale是缩放。

```js
ctx.save(); // 保存canvas 当前状态
ctx.translate(200, 175); // 比 rotate晚执行，先写的后执行
ctx.rotate(10 * Math.PI / 180);
ctx.strokeRect(-100, -75, 200, 150);
ctx.restore(); // 恢复到上一次保存的状态
```

上面的代码，先旋转，绘制，然后再```translate```，```ctx.save()```保存住现在的状态，后面写的改变状态如 ```translate``` 不会产品影响。```ctx.restore()```恢复到上一次保存的```save```状态，类似于初始化。

## 12. 绘制图片

```drawImage(图片, sx, sy, sw, sh, dx, dy, dw, dh)```原图位置大小，目标位置大小。图片对象可以是```img```标签，可以是```new Image```，可以是```canvas```，可以是```video```;

```js
const oImg = new Image(); // 和 document.getElementById('img') 一模一样
oImg.src = 'src';
oImg.onload = function() {
    ctx.drawImage(oImg, 0, 0);
}
```

## 13. 像素操作

像素操作是性能较低的一种操作方式，有跨域的问题，只能读取同域的文件。过程是先获取一块像素，数组然后设置一块像素最后创建一块像素。

```js
const oImg = new Image();
oImg.src = '路径';
oImg.onload = function() {
    ctx.drawImage(oImg, 0, 0);
    const imageData = ctx.getImageData(0, 0, 800, 400);
    // imageData 是一个数组，一个像素占4位，rgba; 四个值都是0 ~ 255
    for (let r = 0; r < h; r++) {
        for (let c = 0; c < w; c++) {
            // (r * w + c) * 4 + 0 红
            // (r * w + c) * 4 + 1 绿
            // (r * w + c) * 4 + 2 蓝
            // (r * w + c) * 4 + 3 透明度
            imageData.data[(r * w + c) * 4 + 0] = 0;
        }
    }
}
ctx.putImageData(imageData, 0, 0)
```

## 14. 视频滤镜

```js
const oV = document.getElementById('video');

// 视频时间变化事件
// oV.addEventListener('timeupdate', () => {
    // ctx.drawImage(oV, 0, 0);
// })
requestAnimationFrame(() => {
    ctx.drawImage(oV, 0, 0);
    const imageData = ctx.getImageData(0, 0, w, h);
    ctx.putImageData(oV, 0, 0);
})
```

## 15. 保存图片

通过```toDataURL()```将图片转为```base64```，然后将图片发给服务器。

```js
http.createServer((req, res) => {
    if (req.url == '/upload_base64') {
        const arr = [];
        req.on('data', data => {
            arr.push(data);
        });
        req.on('end', () => {
            const buffer = Buffer.concat(arr);
            fs.writeFile('1111.png', buffer.toString().replace(/^data:[^,]+;base64,/, ''), 'base64', err => {
                res.end('OK');
            })
        })
    }
});
```

服务器通过处理将```base64```转换为图片流格式，进行下载，响应头设置为```'Content-Disposition', 'attachment; filename=download.png'```

```js
http.createServer((req, res) => {
    if (req.url == '/upload_base64') {
        const arr = [];
        req.on('data', data => {
            arr.push(data);
        });
        req.on('end', () => {
            const buffer = Buffer.concat(arr);
            res.setHeader('Content-Disposition', 'attachment; filename=download.png');
            const data = new Buffer(buffer.toString().replace(/^data:[^,]+;base64,/, ''), 'base64');
            res.end(data);
        })
    }
});
```

```canvas```是位图，不保留图形信息，不能修改，无事件，性能特别高，适合游戏和大型图表，```svg```是矢量图，保留图形信息，能修改有事件，性能和普通标签差不多，适合交互频繁和普通的图表。
