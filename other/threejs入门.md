## 1. 概述

```WebGL```可以理解为原生的```javascript```，而```three.js```更像是```jq```。

```WebGL```可以将```javascrit```与```openGL ES2```结合进行显卡操作，让前端可以通过编写```js```代码，借助显卡在浏览器端编写```3D```图形。不过在这个过程对于图形渲染需要使用着色器语言就是```LES```才能完成页面展示操作。

```Three.js```是一个类库，他的内部提供了很多````3D````显示功能，首先介绍一下几个核心的概念。

- 场景

场景就是所展示在的空间。也可以说是一个舞台。任何要展示的内容都可以放置在场景中。场景就是一个三维空间。

- 相机

相机可以理解为眼睛，可以把它想象成浏览器的眼睛。

相机用来生成快照，将场景和场景中的内容进行快照保存，渲染到页面中。

在```Three.js```中有正投影相机和透视相机，正投影相机会将远处和近处的内容做同等大小的处理，透视相机则更符合人的心理习惯，近大远小。

- 渲染器

渲染器决定了内容如何呈现至屏幕，也就是将相机中的内容渲染到页面。

- 几何体

几何体就是要渲染的图形。

## 绘制一个3D立方体

```js
// 场景构建
const scene = new THREE.Scene();
// 相机(眼睛) - 使用透视相机
// 视角45度，纵横比, 近点距离，远点距离
const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000); // 1到1000米
// 渲染器
const renderer = new THREE.WebGLRenderer({
    antialias: true
});
renderer.setClearColor(0xffffff);
renderer.setSize(window.innerWidth, window.innerHeight);
// 几何体
document.body.appendChild(renderer.domElement);
// 创建几何体
const geometry = new THREE.BoxGeometry(1, 1, 1);
// 材质
const material = new THREE.MeshBasicMaterial({
    color: 0x324334,
    wireframe: true,
});

const cube = new THREE.Mesh(geometry, material);

scene.add(cube);

// 修改一下相机的位置
camera.position.z = 4; // z轴方向提升4米

function animate() {
    requestAnimationFrame(animate);
    cube.rotation.y += 0.1; // 物体旋转
    renderer.render(scene, camera);
}
```

![未命名.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92d20609eedc4273901a9da947a137b8~tplv-k3u1fbpfcp-watermark.image)

## 材质和相机控制

```js
// 定义全局变量
let scene, camera, geometry, mesh, renderer, controls

// 初始化渲染器
function initRenderer() {
    renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setSize(window.innerWidth, window.innerHeight)
    renderer.setPixelRatio(window.devicePixelRatio)
    document.body.appendChild(renderer.domElement)
}

// 初始化场景
function initScene() {
    scene = new THREE.Scene()
    const axesHelper = new THREE.AxesHelper(100)
    scene.add(axesHelper)
}

// 初始化相机
function initCamera() {
    camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)
    camera.position.set(0, 0, 15)
    controls = new THREE.TrackballControls(camera, renderer.domElement)
}

// 初始化模型
function initMesh() {
    geometry = new THREE.BoxGeometry(2, 2, 2)
    // material = new THREE.MeshNormalMaterial()
    const texture = new THREE.TextureLoader().load('./img/crate.gif')
    material = new THREE.MeshBasicMaterial({
    map: texture,
    side: THREE.DoubleSide
    })
    mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)
}

// 初始化动画
function animate() {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
}

// 定义初始化方法
function init() {
    initRenderer()
    initScene()
    initCamera()
    initMesh()
    animate()
}

init()
```

![屏幕快照 2021-06-27 18.31.40.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18f74515355c498f966ee6fadaa94a8c~tplv-k3u1fbpfcp-watermark.image)

## 光源

```js
// 定义全局变量
let scene, camera, geometry, mesh, renderer, controls

// 初始化渲染器
function initRenderer() {
    renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setSize(window.innerWidth, window.innerHeight)
    renderer.setPixelRatio(window.devicePixelRatio)
    document.body.appendChild(renderer.domElement)
}

// 初始化场景
function initScene() {
    scene = new THREE.Scene()
    const axesHelper = new THREE.AxesHelper(100)
    scene.add(axesHelper)

    // const directionalLight = new THREE.DirectionalLight('red')

    // const ambientLight = new THREE.AmbientLight('orange')

    // const pointLight = new THREE.PointLight('green')

    // const spotLight = new THREE.SpotLight('lightblue')

    const hemisphereLight = new THREE.HemisphereLight('red')

    hemisphereLight.position.set(0, 30, 0)
    scene.add(hemisphereLight)

}

// 初始化相机
function initCamera() {
    camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)
    camera.position.set(0, 0, 15)
    controls = new THREE.TrackballControls(camera, renderer.domElement)
}

// 初始化模型
function initMesh() {
    geometry = new THREE.SphereGeometry(3, 26, 26)
    const texture = new THREE.TextureLoader().load('img/crate.gif')
    material = new THREE.MeshPhongMaterial({
    map: texture,
    side: THREE.DoubleSide
    })
    mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)
}

// 初始化动画
function animate() {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
}

// 定义初始化方法
function init() {
    initRenderer()
    initScene()
    initCamera()
    initMesh()
    animate()
}

init()
```

![屏幕快照 2021-06-27 18.38.04.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ef13cbb89c0458e87562be2e21d8b23~tplv-k3u1fbpfcp-watermark.image)

