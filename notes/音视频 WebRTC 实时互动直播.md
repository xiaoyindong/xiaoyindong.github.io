## 1. 概述

首先```WebRTC```是谷歌于```2011```年开源的一个音视频处理引擎，这个引擎是支持跨平台的可以在各个平台编译运行，谷歌希望```WebRTC```用作浏览器之间实现音视频通话这种快速的开发使用的。

```WebRTC```有两个主要功能，一是实时数据传输，大家可能对实时数据传输没什么印象，做通信的同学认知的会深一些，如果在```100ms```的延迟传输，说明通话质量非常好，如果是```200ms```说明通话质量比较优质，如果```500ms```可以接受，如果超过```1s```通话会有明显的迟滞。实时传输是端与端之间建立一条最高效的传输通道，这一点```WebRTC```做的是非常好的。

另一个主要功能就是音视频引擎，引擎并不是简单地做了音视频的编解码，它支持扩展编解码，音视频同步，数据平滑处理。

在实时数据传输，数据处理和异常处理等方面```WebRTC```做的都是比较优秀的。

```WebRTC```不仅用于浏览器，音视频会议，在线教育，照相机，音乐播放器，共享远程桌面，录制，即时通信工具，p2p网络加速，文件传输工具，游戏，实时人脸识别等都可以用到```WebRTC```。并且主流浏览器对```WebRTC```支持都比较不错。

谷歌给提供了一个```demo```地址，可以访问看下，```https://appr.tc/```。

## 2. WebRTC的运行机制

这里要介绍下```轨```和```流```，```轨```就类似火车的轨道，每一条轨道都是独立的，音频是一个轨道，视频也是一个轨道，他们之间是不想交的，会单独存放。

```流```指的是媒体流，和传统的媒体流基本是一个概念，媒体流里面包含音频轨，视频轨还有字幕轨，类似层级的概念，媒体流里面包含了很多轨。

```MediaStream```是媒体流的基础类，```RTCPeerConnection```是整个```WebRTC```里面最重要的一个类包含了很多功能，只需要创建这个实例，将媒体流塞进去就可以了，传输是默认实现了的。```RTCDataChannel```是非音视频的数据。

## 3. 基于node搭建一个https服务

```s
brew install mkcert
// 安装根证书
mkcert ---install
// 生成本地签名，假设域名为123.com
mkcert 123.com
```

使用```https```模块创建```node```服务

```js
const https = require('https');
const fs = require('fs');
const app = https.createServer({
    key: fs.readFileSync('./xxxx.key'),
    cert: fs.readFileSync('./xxxx.pem')
}, (req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('https');
}).listen(443, '0.0.0.0');
```

## 4. WebRTC设备管理

在```WebRTC```规范中给提供一个叫做```enumerateDevices```的```api```可以获取到电脑中的音频和视频设备。

```js
var ePromsie = navigator.mediaDevices.enumerateDevices();
```

返回的是一个```Promise```，在```Promsie```中存在一个```MediaDevicesInfo```，里面存在四个主要的信息。

| 属性 | 说明 |
| ---- | ---- |
| deviceId | 设备ID |
| label | 设备的名字 |
| kind | 设备的种类 |
| groupId | 两个设备groupID如果相同说明是同一个物理设备 |


首先需要判断是否支持```WebRTC```，如果支持就调用```enumerateDevices```方法，```then```中返回的````deviceInfos````是一个数组，每一项就是设备信息。里面包含音频设备和视频设备。

注意这里需要在服务器环境访问，并且需要```https```才可以。

```js
if (navigator.mediaDevices || navigator.mediaDevices.enumerateDevices) {
    navigator.mediaDevices.enumerateDevices().then((deviceInfos) => {
        deviceInfos.forEach((deviceInfo) => {
            console.log(deviceInfo.kind, deviceInfo.label, deviceInfo.deviceID, deviceInfo.groupId);
        })
    }).catch((err) => {
        console.error(err);
    })
} else {
    console.log('不支持这个');
}
```

## 5. 采集音视频数据

使用```getUserMedia```进行数据采集, 同样返回一个```promise```，参数是```MediaStreamConstraints```类型。

```js
var ePromsie = navigator.mediaDevices.getUserMedia(constraints);
```

```getUserMedia```获取视频，传递给```video```标签实时播放。

```html
<video autoplay playsinline id="player"></video>
```

```js```部分同样需要做下判断，```getUserMedia```传入的是```constraints```是一个对象，有两个参数，分别是```video```和```audio```,```video```和```audio```可以是布尔值，也可以是具体音视频的配置，设置```true```，表示音视频都采集。

```then```方法中会获取到流数据，由于这里设置了视频和音频，所以他包含视频轨和音频轨。这里将获取到的流数据给到```video```标签需要使用```html```的```srcObject```属性。

```js
if (navigator.mediaDevices || navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true
    }).then(((stream) => {
        document.querySelector('#player').srcObject = stream;
    }) => {
        
    }).catch((err) => {
        console.error(err);
    })
} else {
    console.log('不支持这个特性');
}
```

## 6. 浏览器兼容问题

```webrtc1.0```规范出来之前，各个浏览器厂商都在按照自己的计划使用```webrtct```这就造成了各个浏览器厂商使用的```getUserMedia```的名字是不一样的都增加了一个自己的前缀。

在```3w```规范里采集音视频数据使用的是```getUserMedia```, 不过在谷歌浏览器里面实现的名字是```webkitGetUserMedia```很像```css3.0```的兼容方式，火狐在前面加的就是```mozGetUserMedia```。

这就给前端开发人员造成了很大的麻烦，如果想通过这个```api```采集音视频数据，对于不同的浏览器厂商就要做类型判断。

```js
const getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;
```

随着```webrtc```在各个浏览器中的推进，```google```开源了一个[adapter.js](https://appr.tc/js/adapter.js)库，这个库就是适配各个浏览器中不同```api```的。最初只有几十行代码，但是随着```webrtc```的发展，现在发展到差不多有两千多行的代码。不过随着时间的推移，浏览器厂商的兼容性越来越好```adapter.js```可能后面会被放弃掉。不过目前来说还是需要使用它来做兼容的。实际工作中最好还是使用他来做兼容。

使用```adapter```很简单，只需要引入这个```js```就可以了, 引入之后在各种浏览器里就可以使用了，包括移动端。

```html
<video autoplay playsinline id="player"></video>
<script src="https://appr.tc/js/adapter.js"></script>
```

不同浏览器中通过```enumerateDevices```获取到的设备信息是不同的，在```chrome```中可以直接获取到设备信息当然这也和不用的版本有关，但是在```苹果浏览器```和```firefox```浏览器中对设备的权限控制会严格一些。

使用```getUserMedia```采集数据的时候浏览器会弹出一个窗口，询问是否允许访问音视频设备。用户点击允许之后才获取到了权限，这个时候再去使用```enumerateDevices```获取设备信息。

## 7. 音视频采集的约束

通过约束可以精确的控制音视频的采集数据。

首先就是宽和高，也就是视频数据的宽高，```width```和```height```，一般视频的宽高有两种比例，一种是```4:3```一种是```16:9```, 比如```320 * 240```，```640 * 480```这都属于```4:3```的比例，显得更方正。```1280 * 720```这是```16:9```的比例，他显得更长一些。

对于手机来说他是反的，比如说如果竖屏拍摄的话那高度是```16```，宽度变成了```9```。通过宽高的约束就可以控制分辨率。

还有比例```aspectRatio```，在这里是个小数点，一般情况下只需要设置宽和高就可以了，一般不会设置这个值。

```frameRate```是帧率可以通过他来控制码流，如果帧率低画面不会平滑，会有一些卡顿，帧率高会很平滑一般```30 - 60```就相对较好了。帧率大码流也会比较大，也就是采集的数据比较多。

```facingMode```这个一般对手机来说，他是控制摄像头翻转的，就是前置摄像头和后置摄像头。```user```是前置摄像头，```environment```是后置摄像头，l```eft```是前置左摄像头，```right```是前置右摄像头。在```PC```这个设置一般没什么作用。

```resizeMode```表示是否裁剪画面，这个用途也不是很多。

这些舒适性的设置比较简单，可以把```video```的布尔值改为对象，然后在里面设置对应的参数。比如宽高设置```640```和```480```，帧率设置```60```，如果是手机可以使用```facingMode```修改使用摄像头。

```js
if (navigator.mediaDevices || navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices.getUserMedia({
        video: {
            width: 640,
            height: 480,
            frameRate: 60,
            facingMode: 'environment'
        },
        audio: false,

    }).then(((stream) => {
        document.querySelector('#player').srcObject = stream;
    }) => {
        
    }).catch((err) => {
        console.error(err);
    })
} else {
    console.log('不支持这个特性');
}
```

同样在音频中也有一些参数约束，首先是```volume```是音量相关的。值是```0-1```。第二个是```sampleRate```采样率，在音频中有很多采样率，比如```48000```，```32000```，```16000```，```8000```等很多，可以根据自己的需要设置采样率。

```sampleSize```还有采样大小或者位深，也就是每一个采样由多少位表示，一般都用```16```位也就是两个字节表示。

```echoCancellation```回音，也就是采集数据后是否开启回音消除，在实时直播的过程中回音消除是一个特别重要的功能。当双方通话的时候如果有回音传过来会对通话质量造成很大的影响。这个参数的值是布尔，可以设置开启或者关闭。

```autoGainControl```表示在原因的声音基础上是否增加音量，也是一个布尔值。

```noiseSuppression```表示降噪，就是在采集数据的时候是否要开启降噪功能。

```latency```是直播过程中的延迟大小，如果设置的小的话代表实时通信的时候延迟性就小。当网络不是特别好的时候，延迟设置的小就会卡顿。

```channelCount```是单声道还是双声道，一般单声道就够了，如果是对乐器来说会使用双声道，这样音质更好。

```deviceID```是如果多个输入输出设备的时候可以进行设备的切换，比如手机前置摄像头和后置摄像头。

```groupID```物理设备

```WebRTC```约束也可以像下面这样写，可以根据网络情况自动的选择。

```js
{
    audio: {
        noiseSuppression: true,
        echoCancellation: true,
    },
    video: {
        width: {
            min: 300,
            max: 640,
        },
        height: {
            min: 300,
            max: 480
        },
        frameRate: {
            min: 15,
            max: 30
        }
    }
}
```

## 8. 处理获取到的视频

可以给视频添加一些特效，因为是在浏览器当中，所以需要使用浏览器的```css filter```。具体支持下表这些特效。

| 特效 | 说明 | 特效 | 说明 |
| ---- | ---- | ---- | ---- |
| grayscale | 灰度 | opacity | 透明度 |
| sepia | 褐色 | brightness | 亮度 |
| saturate | 饱和度 | constrast | 对比度 |
| hue-rotate | 色相旋转 | blur | 模糊 |
| invert | 反色| drop-shadow | 阴影 |

使用也很简单，就是给```video```标签设置```filter```样式。

```css
.blur {
    -webkit-filter: blur(3px);
}
.grayscale {
    -webkit-filter: grayscale(1);
}
.invert {
    -webkit-filter: invert(1);
}
```

截取视频中的某一帧，做法也非常的简单，就是利用```canvas```获取当前播放的帧，最终输出成一张图片就可以了。可以在一个点击事件中做这件事，点击之后，获取```canvas```的```2d```画布，然后通过```drawImage```将视频(```video```标签)绘制到```canvas```中。这时```canvas```中会绘制出当前```video```展示的内容。可以右键另存图片。也可以通过服务端将图片生成下载。

```js
btn.onclick = function() {
    canvas.getContext('2d').drawImage(video, 0, 0, canvas.width, canvas.height)
}
```

```mediastream```方法和事件

```MediaStream.addTranck()```: 向流媒体中加入音视频轨。

```MediaStream.removeTrack()```: 从媒体流中移除指定的轨。

```MediaStream.getVideoTracks()```: 获取所有的视频轨。

```MediaStream.getAudioTracks()```: 获取所有音频轨。

```MediaStream.stop()```: 将媒体流关闭，会关闭每一个轨中的```stop```

```MediaStream.onaddtrack```：添加媒体轨的事件。

```MediaStream.onremovetrack```: 移除媒体轨的事件。

```MediaStream.onended```: 当流结束的时候的事件。

```js
if (navigator.mediaDevices || navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices.getUserMedia({
        video: {
            width: 640,
            height: 480,
            frameRate: 60,
            facingMode: 'environment'
        },
        audio: false,

    }).then(((stream) => {
        document.querySelector('#player').srcObject = stream;
        // 获取第一个视频轨，一般这里只有一个视频轨
       const track = stream.getVideoTracks()[0];
       console.log(track.getSettings()); // 获取视频配置。
    }) => {
        
    }).catch((err) => {
        console.error(err);
    })
} else {
    console.log('不支持这个特性');
}
```

## 9. 录制介绍

录制媒体流实际上就是获取通过```getUserMedia```获取的实时音视频数据。

```MediaRecoder```有很多的事件和方法。使用也非常简单。直接实例化就可以了。

```js
new MediaRecorder(stream, [, options]);
```

这里的参数```stream```是通过```getUserMedia```或者```video```或者```audio```或者```canvas```获取的```stream```。

存在很多的选项。主要有```mimeType```指定录制的是音频还是视频，录制的格式是什么。

格式有很多比如谷歌的音视频格式```video/webm```,```audio/webm```， 还可以指定视频的编码```video/webm;codecs=vp8```,```video/webm;codecs=h264```, 音频编码```audio/webm;codecs=opus```。

```audioBitsPerSecond```是音频的码率，码率根据编码决定有的是```64k```有的是```128k```，```videoBitsPerSecond```视频码率设置的越多清晰度越高比如```720```可能就是```2M```，```bitsPerSecond```是整体码率。

``MediaRecorder``的```api```也比较多```MediaRecorder.start(timeslice)```是开启录制，```timeslice```是可选参数，如果不设置会存储在一个大的```buffer```中，如果设置了这个参数就会按照时间段存储数据，比如说```10s```存储一块数据。

```MediaRecorder.start()```是关闭录制，当停止录制时会触发```dataavailable```事件，获取得到最终的```blob```数据，```MediaRecorder.pause()```是暂停录制，```MediaRecorder.resume()```是恢复录制，```MediaRecorder.isTypeSupported()```是检查录制支持的文件格式。

```ondatavailable```当数据有效会触发，获取```e.data```。这个事件跟随```timeslice```执行，如果没有指定则记录整个数据。如果指定了会定时触发。

```onerror```在出现错误的时候触发录制会自动停止。

```js```中有```4```种数据存储方式```字符串```，```blob```是一个高效的存储区域，```buffer```，```arraybuffer```就是```blob```依赖的底层，可以说```blob```是对```arraybuffer```的封装。```arraybufferview```是各种各样类型的```buffer```。

一般常用```blob```，如果更底层一点可以使用```arraybuffer```。

## 10. 录制案例

这里来演示开始录制，播放录制的视频，下载录制的视频。

```html
<video playsinline id="player"></video>
<video playsinline id="recplayer" controls=“true”></video>
<button id="record">开始录制</button>
<button id="recplay">开始播放</button>
<button id="download">下载</button>
```

点击开始录制按钮的时候，需要判断是否已经开始录制，定义一个```textContent```状态来判断。如果是播放就暂停，如果是暂停就播放。

当开始录制时创建一个```startRecord```函数，并且调用，在```startRecord```函数中首先重置存储录制数据的数组，这是为了避免上一次录制的数据干扰，然后使用```new MediaRecorder```创建录制对象。在录制对象的``ondatavailable``事件中可以不断的获取到录制的视频流，通过```handleDataAvailable```来接收，然后将它拼接在```buf```的采集数组中。

```new MediaRecorder```传入的第一个参数是```getUserMedia```中的```stream```。

在```stopRecord```函数中只需要调用```mediaRecorder.stop```就可以了。

```js
// 初始化一个mediaRecorder录制对象
var mediaRecorder;
// 创建一个存储数据流的数组。
var buf = [];

btnRecord.onclick = () => {
    if (this.textContent === 'start record') {
        startRecord();
        this.textContent = 'stop record';
    } else {
        stopRecord();
        this.textContent = 'start record'
    }
}

function startRecord() {
    // 开始录制时重置数组
    buf = [];
    // 约束视频格式
    const options = {
        mimeType: 'video/webm;codecs=vp8'
    }
    // 判断是否是支持的mimeType格式
    if (!MediaRecorder.isTypeSupported(options.mimeType)) {
        console.error('不支持的视频格式');
        return;
    }
    // 从window中获取stream
    try {
        mediaRecorder = new MediaRecorder(window.stream, options);
        // 处理采集到的事件
        mediaRecorder.ondatavailable = handleDataAvailable;
        // 开始录制
        mediaRecorder.start(10);
    } catch (e) {
        console.error(e);
    }
}

// 处理采集到的数据
function handleDataAvailable(e) {
    if (e && e.data && e.data.size > 0) {
        // 存储到数组中
        buf.push(e.data);
    }
}

function stopRecord() {
    mediaRecorder.stop();
}

if (navigator.mediaDevices || navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices.getUserMedia({
        video: {
            width: 640,
            height: 480,
            frameRate: 60,
            facingMode: 'environment'
        },
        audio: true,
    }).then(((stream) => { // 存储stream到window上
        window.stream = stream;
    }).catch((err) => {
        console.error(err);
    })
} else {
    console.log('不支持这个特性');
}
```

开始播放事件中创建一个```blob```, 传入的第一个参数是```buffer```，第二个参数指定类型也就是说明```blob```中传入的是什么东西，这里指定```video/webm```。这样就生成了一个可以处理```video```的```buffer```的```blob```。

然后将```blob```赋值给```video```标签的```src```属性，这里需要使用```URL```的```createObjectURL```方法来实现。至于```srcObject```属性是赋值直播流的，这里不需要赋值为```null```就可以了。然后使用```play```方法开始播放。

```js
btnPlay.onclick = function() {
    var recplayer = document.getElementById('recplayer');
    const blob = new Blob(buf, { type: 'video/webm'});
    recplayer.src = window.URL.createObjectURL(blob);
    recplayer.srcObject = null;
    recplayer.play();
}
```

这样就可以播放录制的视频了，先开始录制，录制一段时间然后暂停，会自动保存录制的视频，然后再开始播放。就会播放刚刚录制的视频了。

下载和播放类似首先需要拿到录制的数据，同样的也是使用```URL```对象的```createObjectURL```方法创建```url```。然后再创建一个```a```标签，将```url```赋值给```href```属性，设置文件的名称为```aaa.webm```。最后触发```a```的点击事件。

```js
btndownload.onclick = function() {
    const blob = new Blob(buf, { type: 'video/webm'});
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.style.display = 'none';
    a.download = 'aaa.webm';
    a.click();
}
```

## 11. 通过WebRTC捕获桌面

使用```getDisplayMedia```来实现，他与```getUserMedia```是很类似的, 包括参数也是一致的。

```js
var ePromsie = navigator.mediaDevices.getDisplayMedia(constraints);
```

这个功能是```chrome```的实验功能，只在最新的几个版本中存在。需要手动设置一下。

```s
chrome://flags/#enable-experimental-web-platform-features
```

在这个设置中, 将它选中```enabled```来设置打开。

js代码基本就是之前的代码，只需将```getUserMedia```修改为```getDisplayMedia```即可。

```js
if (navigator.mediaDevices || navigator.mediaDevices.getDisplayMedia) {
    navigator.mediaDevices.getDisplayMedia({
        video: true,
        audio: false
    }).then(((stream) => {
        document.querySelector('#player').srcObject = stream;
    }) => {
        
    }).catch((err) => {
        console.error(err);
    })
} else {
    console.log('不支持这个特性');
}
```

打开页面就会弹出共享屏幕，询问是录制整个屏幕还是录制应用窗口，还可以是```chrome```的一个标签页。

## 12. Socket.io处理消息简述

```WebRTC```中对服务器是没有规定的，主要是对web端的一些规定，这主要是因为每个公司的业务模型是不一样的，很难统一这种规范，所以不如干脆让每个公司自己去定义，只要可以实现数据交互就可以了，这样做也比较灵活更容易被接受。

如果没有信令服务器的话```WebRTC```之间是没办法通信的。发起端和接收端想要传递数据的话，有两个信息是必须经过信令服务器相互交换之后才能进行通信的，第一个是媒体信息，也就是如果要实现通信就要确定编解码器，比如说```A```的视频编码是```H264```,```B```也要告诉```A```是否接受```H264```,所以这个信息是必须要传递的。第二个要传递的信息是网络信息，两个客户端尽可能要会选择```p2p```传输，在链接之前如何发现对方，也是通过服务器。

这里简单演示一个通过```socket.io```搭建的服务器。

```s
npm install socket.io --save-dev
```

```node```代码将```https```和```socket```中进行一个绑定。```socketIo = listen(https_server);```。

```js
const http = require('http');
const https = require('https');
const fs = require('fs');
const express = require('express');
const serveIndex = require('serve-index');
const socketIo = require('socket.io');

const app = express();
app.use(serveIndex('./public'));
app.use(express.static('./public'));

const http_server = http.createServer(app);

http_server.listen(80, '0.0.0.0');

const https_server = https.createServer({
    key: fs.readFileSync('./xxxx.key'),
    cert: fs.readFileSync('./xxxx.pem')
}, app);

const io = socketIo.listen(https_server);

https_server.listen(443, '0.0.0.0');

```

绑定之后就可以去处理站点上所有的```connection```事件了，回调函数中的参数socket代表每一个客户端。

首先这台服务器是要有房间的概念，要有加入房间的事件和离开房间的事件。这里加入定义为```join```，离开定义```leave```。客户端传递一个参数说明加入到哪个房间，用```room```来接收。

当收到加入房间的时候，加入到对应的房间，```socket.io```自身就提供了```join```方法也就是加入到那哪个房间。这里的房间就是一个名字或者```id```，如果不存在```socket.io```会自动创建一个房间。```io.sockets.adapter.rooms```, 这是一个对象，可以通过```room```的标识查到。

加入成功之后可以给当前用户回复，也可以给所有人回复。

```js
const io = socketIo.listen(https_server);

io.sockets.on('connection', (socket) => {
    socket.on('join', (room) => {
        socket.join(room);
        const myRoom = io.sockets.adapter.rooms[room];
        const users = Object.keys(myRoom.sockets).length; // 获取房间内用户数量。
        // 当前用户回复
        socket.emit('joined', room, socket.id);
        // 给房间内除了自己所有人回复
        socket.to(room).emit('joined', room, socket.id);
        // 给房间所有人回复
        io.in(room).emit('joined', room, socket.id);
        // 给除了自己的所有人发送
        socket.broadcast.emit('joined', room, socket.id);
    })
})

https_server.listen(443, '0.0.0.0');
```

用户离开的逻辑和用户加入的基本一样，用户数这里需要减一，使用```socket.leave```方法让用户离开房间。

```js
io.sockets.on('connection', (socket) => {
    socket.on('leave', (room) => {
        
        const myRoom = io.sockets.adapter.rooms[room];
        let users = Object.keys(myRoom.sockets).length; // 获取房间内用户数量。
        users -= 1;
        socket.leave(room);
        // 当前用户回复
        socket.emit('joined', room, socket.id);
        // 给房间内除了自己所有人回复
        socket.to(room).emit('joined', room, socket.id);
        // 给房间所有人回复
        io.in(room).emit('joined', room, socket.id);
        // 给除了自己的所有人发送
        socket.broadcast.emit('joined', room, socket.id);
    })
})
```

```H5```端链接socket.io。

```js
// 链接服务
const socket = io.connect();

// 使用on接收消息
socket.on('joined', (room, id) => {

})

// 发送消息, 名字叫join，值为1
socket.emit('join', '1');
```

## 13. 端对端链接

```RTCPeerConnection```是```WebRTC```的核心类, 接收一个可选参数。

```js
new RTCPeerConnection([configuration])
```

类的方法有按功能可以分为四类，媒体协商类，媒体流和轨道类，传输相关类和统计相关类。

对于```AB```两个端来说如果想要创建连接，首先```A```会创建一个```offer```，创建```offer```实际上就形成了一个```sdp```，他是一个包含了媒体信息编解码信息传输的相关的信息，创建之后通过云端的信令牌服务器传给```B```, 在传输之前```A```需要调用```setLocalDescription```方法去收集候选者也就是可连接端。

```B```端收到```offer```之后，会调用```setRemoteDescription```方法将```offer```的```sdp```数据放到自己远端的描述信息槽里，当这些做完之后需要返回```A```一个```answer```，就是创建```B```本身的一个媒体信息，```offer```是```A```的媒体信息，```answer```是```B```的媒体信息，也就是编解码信息之类的。形成之后```B```也要调用一个```setLocalDescription```方法，也是收集候选者。调用之后```B```将```answer```通过服务转给````A````。

```A```收到```answer```也会将这个值使用```setRemoteDescription```存在自己的远程槽里，这样在每一端都有两个```SDP```，第一个是自己的媒体信息，第二是对方的媒体信息。拿到这两个媒体信息之后在内部进行一个协商，比较两者是否可以通信。协商之后取出交集，协商过程就建立好了。

简单来说就是发送端和接收端都有两个数据要设置，自身的数据和另一方的数据。发送端创建数据之后要将数据设置在自己身上还要发送给接收端，接收端拿到之后设置到自己身上，然后接收端创建的数据也要设置在自己身上还要传给发送端，让他也设置到自己身上。

当一开始创建```RTCPeerConnection```的时候协商处于一个```stable```状态，这个时候```connect```就可以用了，但是这时他是不能进行编解码的，因为还没有进行数据的协商。创建```offer```之后再调用```setLocalDescription```将```offer```放进去这个时候状态会变成```hava-local-offer```。

接收到```answer```之后再将```answer```放入```setRemoteDescription```，状态会回到```stable```状态，这个时候```connect```就可以用了，并且是协商过的。

对于被调用者也是一样收到```offer```之后，将```offer```放入到``setRemoteDescription``，状态变成```hava-remote-offer```, 自己创建了```answer```放入```setLocalDescription```中之后状态变成了```stable```。

```createOffer```调用方创建一个```offer```也就是本地的媒体编解码信息。

```option```参数包括```iceResetart```该选项会重启```ICE```，重新进行```Candidate```的收集。```voiceActivityDetection```是否开启静音检测，默认开启。

```js
aPromise = myPeerConnection.createOffer([options])
```

```createAnswer```接收方需要创建一个```answer```

```js
aPromise = myPeerConnection.createAnswer([options])
```

```setLocalDescription```参数是```createOffer```或者```createAnswer```的结果。

```js
aPromise = myPc.setLocalDescription(sessionDescription)
```

```setRemoteDescription```参数是```createOffer```或者```createAnswer```的结果。


```js
aPromise = myPc.setRemoteDescription(sessionDescription)
```

媒体协商的方法很简单，只有添加和移除。

```addTrack```第一个是要添加的```track```，第二个是流里面可能有音频也可能有视频，需要将他们遍历添加进来，这样就获取到了```PeerConnection```，里面包含要传输的音视频了。

```js
rtpSender = myPc.addTrack(track, stream...)
```

```removeTrack```可以删除轨道

```js
myPc.removeTrack(rtpSender);
```

```onnegotiationneeded```表示协商事件，当进行媒体协商的时候会触发这个事件。

```onicecandidate```当收到一个```ice```候选者的时候会触发这个事件。可以拿到这个候选者将它添加到```ice```中去，这样就可以进行通信了。

这里来介绍一下端对端链接的基本流程，首先这里需要有四个实体，```发起端A```,```接收端B```,```服务器```，```sturn和turn服务```。

首先```A```和```B```要与服务器建立链接，这样就可以实现数据中转, 接着```A```如果想要发起呼叫需要创建一个```PeerConnect```, 通过```getUserMedia```拿到本地的音视频流，将这个流添加到连接中去, 接着可以调用```PeerConnect```的```create offe```r创建一个```offer```的```sdp```，然后调用```setLocalDescription```将它设置到这个槽中去，掉完这个方法会发送一个请求给```STUN/TURN```服务，这时候就开始收集所有可以连接的候选者了。

接着再把```offer```发给服务器，服务器将它转给```B```,```B```就拿到了```offer```，他首先也要创建一个```PeerConnect```，然后调用```setRemoteDescription```，然后再创建一个```answer```，然后调用```setLocalDescription```将它设置进去。他也会发送一个请求给```STUN/TURN```服务，这时候就开始收集所有可以与A通信的候选者。

```B```将```answer```通过服务器转发给```A```,这时```A```再调用```setRemoteDescription```就进行媒体协商了。

````A````拿到候选者之后将数据通过服务器发送给```B```端，让```B```知道有哪些链接通路，```B```将链接通路添加到列表中。同样```B```也通过服务器发送给```A```,让```A```存储到自己的通路添加到列表中。这时```A```和```B```就可以进行通信了。

## 14. 案例演示

这里简单演示一下，在一个页面中创建两个```video```一个展示本地采集的视频，另一个模拟传递过来的视频。由于这里没有```STUN/TURN```服务，就省略掉这个步骤。数据只在本地流转，模拟真实的传输过程。主要是为了演示上面提到的流程。

在页面中新建两个```video```，一个是表示展示本地数据，一个表示展示远端数据。除了两个```video```，还要有```button```, 通过```startbutton```开始采集数据，然后展示在```localvideo```中。

第二个```button```用来创建双方的```connection```，媒体协商之后然后将收到的数据渲染到```remotevideo```中。最后一个```button```是结束。

```html
<video id="localvideo" autoplay palysinline></video>
<video id="remotevideo" autoplay palysinline></video>
<button id="start">start</button>
<button id="call">call</button>
<button id="hangup">hangup</button>
<script src="https://webrtc.github.io/adapter/adapter/adapter-latest.js"></script>
```

点击start的时候需要采集数据，在返回值里面需要做两件事，第一件是设置到````localvideo````中。然后这里要将```stream```存储到全局变量中```localStream```，后面还需要使用。

```js

let localStream;

btnstart.onclick = function start() {
    if (navigator.mediaDevices || navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({
            video: true,
            audio: false
        }).then(((stream) => {
        localVideo.srcObject = stream;
        localStream = stream;
        }).catch((err) => {
            console.error(err);
        })
    } else {
        console.log('不支持这个特性');
    }
}
```

点击```call```的时候逻辑比较复杂，首先要创建```pc1```的```PeerConnection```和```pc2```的```PeerConnection```, 这里是需要一个可选参数的，参数是需要的网络传输的配置整个```ICE```的配置，这里演示没有不涉及网络，所以就不设置这个参数了。

拿到两个链接之后需要添加一些事件，需要知道```candidate```收集到之后的事情，最主要的也是这个事件。在这个事件中需要通过服务器将数据传回给```pc2```。所以在```pc1```的```onicecandidate```事件中调用```pc2```的```addIceCandidate```方法。对于```p2```同样监听```onicecandidate```，然后交给```pc1```。这是```pc1```和```pc2```收集到```Candidate```之后要做的事情。

```js
var pc1;
var pc2;
call.onclick = function() {
    pc1 = new RTCPeerConnection();
    pc2 = new RTCPeerConnection();

    pc1.onicecandidate = function(e) {
        pc2.addIceCandidate(e.candidate);
    }
    pc2.onicecandidate = function(e) {
        pc1.addIceCandidate(e.candidate);
    }
}
```

对于```pc2```还要有一个```ontrack```，当他收到```track```的时候就接收到了```stream```，所以这里将```streams```放到```remoteVideo```的```srcObject```中。这里有很多的```stream```，取第一个就可以了。这样就将远端的音视频流传给了```remoteVideo```中。

```js
pc2.ontrack = function(e) {
    remoteVideo.srcObecjt = e.streams[0];
}
```

接着要将本地采集的数据添加到```pc1```的```PeerConnection```中去，这样在创建媒体协商的时候才会知道有哪些媒体数据。这个顺序不能乱，是现有数据才能做媒体协商，不能先协商后有数据。

所以要先添加流，这里通过全局的```localStream```的```getTracks```拿到所有的轨，循环他传入的```pc1```的```track```中，第一个参数是当前的```track```，第二个参数是所在的流。这样就将本地采集的音视频流添加到了```pc1```的```PeerConnection```。

```js
localStream.getTracks().forEach(function(track) {
    pc1.addTrack(track, localStream);
})
```

这个时候就可以去创建```pc1```去媒体协商了，第一步是创建```offer```，这里的参数可以指定本地媒体的信息。因为没有采集音频所以```offerToRecieveAudio```是```0```，```offerToRecieveVideo```是```1```。

有了这个就可以创建本地的媒体信息了，他是一个```promise```。可以在```then```中可以拿到描述信息，在这里要调用```pc1```的```setLocalDescription```将他传入进去。

正常完成这步接着应该发送```desc```到信令服务器，信令服务会转发给```B```, 所以第二个人会从信令服务中接收```desc```。然后第二个人会调动```setRemoteDescription```将```desc```设置为自己的远端。

设置之后```pc2```创建自己的```answer```, 这里是不需要传递参数的。创建成功之后也是一个```promise```，在这个```then```中```pc2```会将参数设置为自己的```LocalDescription```。然后```pc2```也开始收集```candidate```，然后他也会将自己的```desc```发送到信令服务。与```pc1```进行交换，```pc1```会接收这个```desc```设置自己的```remoteDescription```

```js
pc1.createOffer({
    offerToRecieveAudio: 0,
    offerToRecieveVideo: 1,
}).then(function(desc) {
    pc1.setLocalDescription(desc);
    // send desc to 信令服务
    // pc2 receive desc from 信令服务
    pc2.setRemoteDescription(desc);

    pc2.createAnswer().then(function(desc2) {
        pc2.setLocalDescription(desc2);
        // send desc2  to 信令服务
        // pc1 receive desc from 信令服务
        pc1.setRemoteDescription(desc2);
    })
})
```

这样整个协商就完成了，并且对```candidate```也收集完了，然后进行交换形成连接列表，然后进行连接检测，检测之后就开始真正的发送了。

当关闭的时候执行```close```方法就可以了。

```js
hangup.onclick = function() {
    pc1.close();
    pc2.close();
    pc1 = null;
    pc2 = null;
}
```

```SDP```它是一种信息格式的描述标准，本身不属于传输协议，但是可以被其他传输协议用来交换必要的信息。最主要的用途就是之前所讲的进行媒体的协商。

## 15. STUN/TURN服务器搭建

市面上有很多的```STUN/TURN```这种服务，一般都是将这两个协议融合在一起，也就是在一个服务中既支持```STUN```协议又支持```TURN```协议。一个比较有名的就是```rfc5766-turn-server```这个```turn-server```最初是由```google```发起的，也有很多人使用，不过他有很多的功能不足，很多人在他之上做了一些修改形成了现在的```coTurn```。```coTurn```是```rfc5766-turn-server```的升级版本。它支持了```UDP```和```TCP```还支持```IPV4```和```IPV6```。这里也选用```coTurn```, 因为他的活跃度比较高，用户量比较大。

还有一种叫做```ResTurn```相对来说他比```coTurn```差一些。

可以在```github```的```coTurn```中[下载](https://github.com/coturn/coturn), 将源码```clone```下来，下载之后需要进行编译。安装到```/usr/local/coturn```目录中。

```js
./configure --prefix=/usr/local/coturn
// -j是表示多线程编译，一般根据CPU核心数的2倍来定，如果双核就用4
make -j 4
// 安装
sudo make install
```

安装之后```/usr/local/coturn```就会存在，```bin```目录下面是可执行的程序，配置文件在```etc```目录下。

对```coturn```来说有很多的配置项，其中主要的就是下面这几项。

```s
listening-port=3378  # 指定侦听的端口，默认3478
external-ip=0.0.0.0  # 指定云主机的公网IP地址
user=aaaaa:bbbbb     # 访问 stun/turn服务的用户名和密码
realm=stun.xxx.cn    # 域名，这个一定要设置
```

配置文件在```/usr/local/coturn/etc/turnserver.conf```,可以在这个文件中进行修改编辑。设置上面四项就足够了。配置之后启动这个服务。

```s
cd /usr/local/coturn/bin
./turnserver -c ./etc/turnserver.conf
```

启动之后可以在这个网站中做下测试```webrtc.github.io/samples/src/content/peerconnection/trickle-ice/```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f7622e442124a188ad3ecdec44019ae~tplv-k3u1fbpfcp-watermark.image)

添加测试的```url```和端口，输入用户名和密码，点击添加服务，然后点击下面的```Gather candidates```开始收集。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c302721f4ce4ea1a126f48374f1425e~tplv-k3u1fbpfcp-watermark.image)

这里有不同的服务和不同的端口，有```IPV4```的地址也有```IPV6```的端口。也有```TCP```也有```UDP```。还有他们的优先级。

```js
new RTCPeerConnection()
```

在之前使用```RTCPeerConnection```的时候是没有添加参数的, 其实它里面可以有很多的参数。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8de8d7c40563456d898a44ba483ef832~tplv-k3u1fbpfcp-watermark.image)

最关键的是```iceServers```也就是```stun/turn```服务，通过这个服务他可以做检测来获取到相应的反射地址和中继地址。有了这些就可以在做连接性检测的时候找到优先级。

第二个是```iceTranportPolicy```, 这是传输策略，他有两种，一种是```all```一种是```relay```，```all```支持反射后的```candidates```和中继的```candidates```，如果是```relay```那就只收集中继的```candidates```。

第三个是```bundlePolicy```这个策略也有好几种默认是```balanced```就是平衡的，后续会详细介绍。

还有一个就是```rtcpMuxPolicy```就是```RTC```的复用策略，默认是```require```。

```peerIdentity```是一个标识的字符串，```certificates```是一些证书，就是每一个链接，每一个可连通的候选者都需要有一个证书，如果有多个链接就会有多个证书，如果复用的情况下有一个证书就可以了，这样可以增加传输速度。

最后一个就是```iceCandidatePoolSize```就是要收集的候选者的空间，默认是```0```，如果设置```5```的话，即使有```20```个也选其中的```5```个。

下面来详细看一下。

### 1. bundlePolicy

balanced如果有多路传输通道音频与视频轨使用各自的传输通道。

max-compat最大兼容性，每个轨道使用自己的传输通道。bundle绑定不成功的时候实际上走的就是这个方式。

max-bundle最大话的使用绑定，都绑定到同一个传输通道。这个是WebRTC建议的方式，这样最简单，而且证书只需要一个，因为每个链接都需要一个证书。

### 2. certificates

授权可以使用链接的一组证书，如果不提供会自动产生，所以一般不会设置。

### 3. iceCandidatePoolSize

```16```位的整数值，用于指定预取的```ICE```候选者的个数，如果该值发生变化，它会触发重新收集候选者。

### 4. iceTransportPolicy

指定```ICE```传输策略，一般设置为```relay```，默认是```all```

```relay```收集候选者的时候只收集中继候选者

```all```可以使用任何类型的候选者

### 5. iceServers

由```RTCIceServer```组成，每个```RTCIceServer```都是一个```ICE```代理的服务。

| 属性 | 含义 |
| ---- | ---- |
| credential | 凭据，只有TURN服务使用 |
| credentialType | 凭据类型可以password或oauth |
| urls | 用于连接服务中的url数组 |
| username | 用户名，只有TURN服务使用 |

### 6. rtcpMuxPolcy

在收集```ICE```候选者时使用

| 选项 | 说明 |
| ---- | ---- |
| negotiate | 收集RTCP与RTP复用的ICE候选者，如果RTCP能复用就与RTP复用，如果不能复用，就将他们单独使用 |
| require | 只收集RTCP与RTP复用的ICE候选者，如果RTCP不能复用，则失败。 |

### 7. candidate

| 属性 | 说明 |
| ---- | ---- |
| candidate | 候选者描述信息 |
| sdpMid | 与候选者相关的媒体流的识别标签 |
| sdpMLineIndex | 在SDP中 m=的索引值 |
| usernameFragment | 包括了远端的唯一标识 |

## 16. 音视频直播客户端实现

首先存在两个按钮，链接和离开，然后有两个```video```标签，一个展示本地的```video```，一个展示远端获取的```video```

当点击链接按钮的时候，需要打开本地的连接设备, 拿到之后将视频渲染到本地```video```中，再存储到```localStream```变量中，这样本地视频就展示出来了。调用```connection```链接函数。

```js

let localStream;
connect.onclick = function() {
    if (navigator.mediaDevices || navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({
            video: true,
            audio: true
        }).then(((stream) => {
            localStream = stream;
            document.querySelector('#local').srcObject = stream;
            // 链接服务端函数
            connection();
        }).catch((err) => {
            console.error(err);
        })
    } else {
        console.log('不支持这个特性');
    }
}
```

当调用```joined```的时候表示第一个人进来了，这时候调用```createPeerConnection```创建链接。当执行```otherjoin```的时候表示房间不只一个人，需要建立媒体协商了。

```js
let socket;
function connection() {
    // 链接服务器
    socket = io.connect();
    // 接收服务端加入成功的消息，接收房间id和用户id
    socket.on('joined', (roomid, id) => {
        createPeerConnection();
    })
    // 其他人加入
    socket.on('otherjoin', (roomid, id) => {
        // 第二个人加入
    })
    // 
    socket.on('full', (roomid, id) => {
        socket.disconnect(); // 断开链接
    })
}
```

创建```PeerConnection```，首先判断有没有创建过，如果没有就创建, 这里传入```iceServers```设置```urls```地址这个前面讲过了，链接的用户名和密码。这样就配置了一个简单的```ice```。

创建链接之后需要监听```onicecandidate```事件和```ontrack```事件。然后需要给本地的```track```获取到，然后添加到```pc```中，告诉对方，自己本身有哪些媒体流，对方会根据自身的媒体流创建相同类型的媒体流。

在```onicecandidate```函数中，每当发现```candidate```就会执行，所以要发送一个消息, 发送给房间中的所有人这个消息是```candidate```。

```js
let pc;
function createPeerConnection() {
    if (!pc) {
        pc = new RTCPeerConnection({
            'iceServers': [{
                'urls': 'turn:sturn.al.aaaaa.cn:3478',
                'credential': 'password',
                'username': 'username'
            }]
        })

        pc.onicecandidate = (e) => {
            if (e.candidate) {
                socket.emit('message', '1111', {
                    type: 'candidate',
                    label: e.candidate.sdpMlineIndex,
                    id: e.candidate.sdpMid,
                    candidate: e.candidate.candidate
                }); // 发送消息
            }
        }
        pc.ontrack = (e) => { // 获取远端的流，赋值给video标签。
            remoteVideo.srcObject = e.streams[0];
        }
    }

    if (localStream) { // 给本地Stream添加到pc中，
        localStream.getTracks().forEach(track => {
            pc.addTrack(track);
        })
    }
}
```

关闭媒体流比较简单，调用```close```方法就可以了。

```js
function closePeerConnection() {
    if (pc) {
        pc.close();
        pc = null;
    }
}
```

创建```offer```，然后发送给对端，这个方法只能在发起端调用。

创建```createOffer```这里先传入音频和视频选项, 表示接收远端的视频和音频，在回调里面，当收到```desc```之后需要调用```setLocalDescription```, 来收集```candidate```。设置之后要给另一端发送一个消息，让对方创建一个```answer```，这里要发送给房间中的另一个人，所以传入房间号```1111```，和```desc```。

```js
function call() {
    if (pc) {
        pc.createOffer({
            offerToReceiveAudio: 1,
            offerToReceiveVideo: 1,
        }).then(function(desc) {
            pc.setLocalDescription(desc);
            socket.emit('message', '1111', desc); // 发送消息
        })
    }
}

```

在```socket```的```message```中需要做一些逻辑, 首先要判断传入进来的是```offer```还是```answer```还是```candidate```，他们的逻辑是不同的。

如果是```offer```，表示粉丝接收到了直播人发来的```offer```，所以他要将这个```desc```设置到```setRemoteDescription```，由于传递过来之后会自动转换成文本，所以需要使用```RTCSessionDescription```将它重新转换为对象,设置之后要创建一个```answer```, 在这个回调中设置到本地，```setLocalDescription```, 然后发送给直播端。

如果数据类型是```answer```，也就是直播人收到了回答，设置到自己的```setRemoteDescription```。

如果类型是```candidate```，需要创建```candidate```, 然后将它添加到本地的```candidate```中。

```js
let socket;
function connection() {
    // 链接服务器
    socket = io.connect();
    // 接收服务端加入成功的消息，接收房间id和用户id
    socket.on('joined', (roomid, id) => {
        createPeerConnection();
    })
    // 其他人加入
    socket.on('otherjoin', (roomid, id) => {
        // 第二个人加入
    })
    // 
    socket.on('full', (roomid, id) => {
        socket.disconnect(); // 断开链接
    })

    socket.on('message', (roomid, data) => {
        if (data.type === 'offer') {
            pc.setRemoteDescription(new RTCSessionDescription(data));
            pc.createAnswer().then((desc) => {
                pc.setLocalDescription(desc);
                socket.emit('message', '1111', desc); // 发送消息
            });
        } else if (data.type === 'answer') {
            pc.setRemoteDescription(new RTCSessionDescription(data));
        } else if (data.type === 'candidate') {
            let candidate = new RTCIceCandidate({
                sdpMlineIndex: date.label,
                candidate: data.candidate,
            })
            pc.addIceCandidate(candidate);
        }
    })
}
```

```getDisplayMedia```可以采集桌面无法同时采集音频。

## 17. WebRTC核心之RTP媒体控制与数据统计

这里介绍```RTP```的```Media```，在这一层是真正处理数据传输的，控制了分辨率，传输规则，帧率，码流等内容。

这里主要有两个类，一个是```Receiver```一个是```Sender```。也就是一个接收类，一个是发送类。

```RTCRtpReceiver```的方法

| 方法 | 说明 |
| ---- | ---- |
| getParameters | 返回RTCRtpParameter 对象 |
| getSynchronizationSources | 返回一组SynchronizationSources实例 |
| getContributingSources | 返回一组ContributingSources实例 |
| getStats | RTCStatsReport,里面包括输入流统计信息 |
| getCapabilities | 返回系统能接收的媒体能力(音频，视频) |

```RTCRtpSender```的方法

| 方法 | 说明 |
| ---- | ---- |
| getParameters | 返回RTCRtpParameter 对象 |
| setParameters | 设置RTP传输相关的参数 |
| getStats | 提供了输出流的统计数据 |
| replaceTrack | 用另一个track替换现在的track，如切换摄像头 |
| getCapabilities | 按类型(音频，视频)返回系统发送媒体的能力 |

下面来控制一下传输速率，来演示一下上面介绍的```sender```和```receive```，控制速率就是在```sender```中控制的。这个设置要在协商之后打开才可以生效，对于发送者来说是收到```answer```的时候，接受者来说是创建```answer```之后。

```js
// 定义vsender变量来存储获取到的sender
let vsender;
const senders = pc.getSenders(); // 获取所有发送器
// 这里控制视频，就不演示音频了，原理一样
// 遍历每一个发送器
senders.forEach(function(sender) {
    // 如果是视频就赋值给vsender
    if (sender && sender.track.king === 'video') {
        vsender = sender;
    }
})
// 获取拿到的发送器的参数
const parameters = vsender.getParmeters();
// 判断encodings是否存在值，因为要修改encodings
if (parameters.encodings) {
    // 设置maxBitrate
    parameters.encodings[0].maxBitrate = 2000 * 1000;
}
// 修改完成后需要再设置回vsender, setParameters是一个promise
vsender.setParameters(paramters).then(() => {
    console.log('设置成功')
}).catch((error) => {
    console.error(error);
})
```

可以在```chrome://webrtc-internals```中查看效果，这是谷歌的一个调试工具。

## 18. WebRTC网络基础补充

### 1. NAT: network address translator，对于网络主机来说必须有个公网地址才可以进行通信，NAT就是将内网的ip映射为外网的ip和端口，这就实现了通信。

```NAT```产生的原因第一个是因为```IPv4```地址不够用，第二个是处于安全的考虑，办公室内所有的主机都不能被外网直接访问，需要经过```NAT```的处理，也就是网关层。

### 2. STUN: 两个公网地址需要经过介绍才能连接到一起，这就是STUN，STUN充当一个中介的作用。

他的作用就是进行```NAT```穿越，他是典型的客户端、服务器模式，客户端发送请求，服务端响应请求。

在```RFC```中有两种```STUN```标准，一种是```RFC3489```, 不过他基于```UDP```进行穿越，失败率较高，```RFC5389```，他是在```3489```的基础上增加了一些功能。成功率提高很多。基于```UDP```和```TCP```。

### 3. TURN: 不是所有的P2P都可以连接成功，当P2P连接不成功的时候引入TURN服务，负责双方之间流媒体的转发，其实是一个云端服务器。

```TURN```出现的目的是解决```NAT```穿过过程无法穿越的问题，在```NAT```无法穿越如何解决，```TURN```服务其实就是在服务端架设一个```TURN```服务，他是建立在```STUN```之上，消息格式使用```STUN```格式消息。

### 4. ICE: 是将NAT，STUN, TURN打包成一体，然后选择其中最优的一条线路，当出现问题时会切换到其他线路。

每个```candidate```是一个地址，包括```IP```，```端口```，```协议```。

## 19. WebRTC核心之SDP详解

```SDP```一般分为两层，会话层和媒体层。

媒体信息包含媒体格式和传输协议，传输```ip```，端口，负载类型。

## 20. WebRTC非音视频数据传输

```WebRTC```除了传输音视频，还可以传输文件，聊天，网络加速器等。下面来详细看下。

传输非音视频文件借助的是```createDataChannel```这个类，返回的也是一个```promise```。他接收两个参数，一个```lable```标识给人看的，一个```options```，```options```也是可选的。

```options```第一个是```ordered```表示是否按顺序到达，因为```UDP```是无序的。```maxPacketLifeTime/MaxRetransmits```这两个参数互斥，表示最大存活时间和尝试次数。

```negotiated```如果是```false```接收方使用```ondatachannel```监听，如果是```true```两端都可以调用```createDataChannel```创建通过通过```id```来标识唯一。

```js
pc.createDataChannel('chat', { negotiated: true, id: 0 });
```

onmessage：接收数据

onopen: 创建好```datachannel```的时候触发这个或者当第一次有消息来的时候触发。

onclose：当```datachannel```关闭的时候

onerror：出错的时候

```js
var pc = new RTCPeerConnection();
var dc = pc.createDataChannel('dc', options);

dc.onerror = (error) => {
    console.log(error);
}

dc.onmessage = (event) => {

}
```

非音视频数据传输方式对比：

| 特性 | TCP | UDP | SCTP(流) |
| ---- | ---- | ---- | ----- |
| 可靠性 | 可靠 | 不可靠 | 可配置 |
| 可达性 | 有序 | 无序 | 可配置 |
| 传输方式 | 字节 | 消息包 | 消息包 |
| 流控 | 需要 | 不需要 | 需要 |
| 网络拥塞控制(丢包，抖动) | 需要支持 | 不需要 | 需要 |

```datachannel```可以传输任何二进制的数据。
