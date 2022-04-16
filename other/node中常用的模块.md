## 1. process

```process```是一个全局的变量，所以使用的时候是不需要执行```require```操作，可以直接使用。

这里分两部分来说明，第一个就是可以借助它去获取进程信息，比如进程工作的时候本地是一个什么样的环境，通过```process```可以获取。第二个通过```process```可以对当前的进程做一些操作，比如说可以监听进程执行过程中内置的事件，创建子进程完成更多的操作。

### 1. 内存相关获取

```js
// 查看内存消耗
console.log(process.memoryUsage());
/**
* rss： 常驻内存
* heapToal: 总内存大小
* heapUsed: 已使用内存
* external: 扩展内存 - 底层模块占用的C/C++核心模块
* arrayBuffers: 缓冲区大小
*/
```

### 2. CPU相关信息获取

```js
console.log(process.cpuUsage());
/**
* user: 用户占用的时间片段
* system: 系统占用的时间片段
*/
```

### 3. 运行时可以通过process查看运行目录，Node环境，cpu架构，用户环境，系统平台。

```js
process.cwd(); // 运行目录
process.version; // Node版本
process.versions; // 运行环境版本
process.arch; // cpu架构
process.env.Node_ENV; // 环境 需要先设置
process.env.PATH; // 环境变量
process.env.USERPROFILE; // 管理员目录路径 不同环境方式不一样 process.env.HOME
process.platform; // 平台 win32 macos

```

### 4. 运行时可以获取启动参数，PID，运行时间，

```js
process.argv; // 获取运行参数，空格分隔可在数组中获取到，默认会存在Node目录和执行脚本的目录两个值。
process.argv0; // 获取第一个值, 只有这一个api
process.pid; // 获取运行的pid
process.ppid; 
process.uptime; // 脚本运行时间
```

事件监听在```process```中提供的内容。这里不会着重说明```process```里面到底有哪些事件，主要还是看一看在```NodeJS```里面熟悉一下事件驱动编程以及发布订阅的模式。

```process```是实现了```emit```接口的。可以使用```on```监听事件，内部提供了很多的事件，比如```exit```，程序退出的时候执行。这里绑定的事件只能执行同步代码，是不可以执行异步代码的，这里要注意。

```js
process.on('exit', (code) => { // 退出时
    console.log(code); // 0
})

process.on('beforeExit', (code) => { // 退出之前
    console.log(code); // 0
})
```

手动退出，这种退出不会执行```beforeExit```，而且```exit```后面的代码也不会执行，因为执行```exit```就已经退出了。

```js
process.exit();
```

### 5. 标准输出，输入，错误

```js
process.stdout; // 是一个流，可以对他进行读写操作。

process.stdout.write('123'); // 123
```

```js
const fs = require('fs');

fs.createReadStream('text.txt').pipi(process.stdout); // 读取文件输出。
```

```js
process.stdin; // 可以拿到控制台输入的内容
process.stdin.pipe(process.stdout); // 输入之后输出
```

```js
// 设置字符编码
process.stdin.setEncoding('utf-8');
// 监听readable事件，是否可读也就是有无内容
process.stdin.on('readable', () => {
    // 获取输入的内容
    let chunk = process.stdin.read();
    if (chunk !== null) {
        process.stdout.write(chunk);
    }
})
```

## 2. path

```Node```中的内置模块，可以直接使用```require```将它导入，他的主要作用就是处理文件的目录和路径。只需要调用不同的方法。```path```相当于一个工具箱，只需要掌握它里面提供的工具，也就是方法。

```js
const path = require('path');
```

### 1. basename()

获取路径中基础名称

```js
path.basename(__filename); // test.js
// 传入第二个参数如果匹配会省略后缀，不匹配仍旧返回真实的后缀
path.basename(__filename, '.js'); // test
path.basename('/a/b/c'); // c
path.basename('/a/b/c/'); // c
```

### 2. dirname()

获取路径中的目录名称

```js
path.dirname(__filename); // d:\Desktop\test
path.dirname('/a/b/c'); // /a/b
path.dirname('/a/b/c/'); // /a/b
```

### 3. extname()

获取路径中的扩展名称

```js
path.extname(__filename); // .js
path.extname('/a/b'); //
path.extname('/a/b/index.html.js.css'); // .css
path.extname('/a/b/index.html.js.'); // .
```

### 4. isAbsolute()

获取路径是否是绝对路径

```js
path.isAbsolute('a'); // false
path.isAbsolute('/a'); // true
```

### 5. join()

拼接多个路径片段，还原成完整可用路径

```js
path.join('a/b', 'c', 'index.html'); // a/b/c/index.html
path.join('/a/b', 'c', 'index.html'); // /a/b/c/index.html
path.join('a/b', 'c', '../', 'index.html'); // a/b/index.html
```

### 6. resolve()

返回一个绝对路径

```js
path.resolve(); // 获取绝对路径
```

### 7. parse()

解析路径

```js
const obj = path.parse('/a/b/c/index.html');
/**
* root: /
* dir: /a/b/c
* base: index.html
* ext: .html
* name: index
*/
```

### 8. format()

序列化路径，与```parse```功能相反, 将对象拼接成完整的路径。

```js
path.format({
    root: '/',
    dir: '/a/b/c',
    base: 'index.html',
    ext: '.html',
    name: 'index'
});
// /a/b/c/index.html
```

### 9. normalize()

规范化路径，将不可用路径变为可用路径, 这个方法注意如果有转译字符会转译。

```js
path.normalize('a/b/c/d'); // a/b/c/d
path.normalize('a//b/c../d'); // a/b/c../d
path.normalize('a\\/b/c\\/d'); // a/b/c/d 
```

## 3. Buffer

```Buffer```一般称为缓冲区，可以认为因为```Buffer```的存在让开发者可以使用```js```操作```二进制```。```IO```行为操作的就是二进制数据。```NodeJS```中的```Buffer```是一片内存空间。他的大小是不占据```V8```内存大小的，```Buffer```的内存申请不是由```Node```生成的。只是回收的时候是```V8```的```GC```进行回收的。

```Buffer```是```NodeJS```中的一个全局变量，无需```require```就可以直接使用。一般配合```stream```流使用，充当数据的缓冲区。

```alloc```可以创建指定字节大小的```Buffer```，默认没有数据

```allocUnsafe``` 创建指定大小的```Buffer```但是不安全，使用碎片的空间创建```Buffer```，可能存在垃圾脏数据，不一定是空的。

```from``` 接收数据创建```Buffer```

在```v6```版本之前是可以通过实例化创建```Buffer```对象的，但是这样创建的权限太大了，为了控制权限，就限制了实例化创建的方式。

```js
// 创建Buffer
const b1 = Buffer.alloc(10);
const b2 = Buffer.allocUnsafe(10);
```

```from```创建```Buffer```可以接收三种类型，字符串，数组，```Buffer```。 第二个参数是编码类型。

```js
const b3 = Buffer.from('1');
const b4 = Buffer.from([1, 2, 3]);
```

```Buffer```的一些常见实例方法。

fill: 使用数据填充```Buffer```，会重复写入到最后一位

write：向```Buffer```中写入数据，有多少写多少，不会重复写入。

toString: 从```Buffer```中提取数据

slice: 截取```Buffer```

indexOf：在```Buffer```中查找数据

copy: 拷贝```Buffer```中的数据

```Buffer```的静态方法。

concat: 将多个```Buffer```拼接成一个新的```Buffer```

isBuffer: 判断当前数据是否是一个```Buffer```

```Buffer```的```split```方法实现。

```js
Array.Buffer.splice = function(sep) {
    let len = Buffer.form(sep).length;
    let ret = [];
    let start = 0;
    let offset = 0;

    while(offset = this.indexOf(sep, start) !== -1) {
        ret.push(this.slice(start, offset))
        start = offset + len;
    }
    ret .push(this.slice(start));
    return ret;
}
```

## 4. fs

在```Node```中```Buffer```和```Stream```随处可见，他们用于操作二进制数据。

```fs```是一个内置的核心模块，所有与文件相关的操作都是通过```fs```来进行实现的，比如文件以及目录的创建，删除，信息的查询或者文件的读取和写入。

如果想要操作文件系统中的二进制数据需要使用```fs```模块提供的```API```，这个过程中```Buffer```和```Stream```又是密不可分的。

介绍```fs```模块之前我们首先需要介绍一下文件系统的基础知识，比如权限位，标识符，文件描述符等。

权限是指当前的操作系统内不同的用户角色对于当前的文件可以执行的不同权限操作，文件的权限操作被分为```r```,```w```,```x```三种, ```r```是读权限，```w```是写权限，```x```是执行权限。如果用8进制的数字进行表示```r```是```4```，```w```是```2```，```x```是```1```，如果不具备该权限就是一个```0```。

操作系统中将用户分为三类分别是文件的所有者，一般指的是当前用户自己，再有就是文件的所属组，类似当前用户的家人，最后是其他用户也就是访客用户。

```Node```中```flag```表示对文件操作方式，比如是否可读可写。

r: 表示可读

w: 表示可写

s: 表示同步

+: 表示执行相反操作

x: 表示排他操作

a: 表示追加操作

```fd```就是操作系统分配给被打开文件的标识，通过这个标识符文件操作就可以识别和被追踪到特定的文件。不同操作系统之间是有差异的，```Node```为我们抹平了这种差异。

```Node```每操作一个文件，文件描述符就会递增一次，并且它是从```3```开始的。因为```0```，```1```，```3```已经被输入，输出和错误占用了。后面我们在使用```fs.open```打开文件的时候就会得到这个```fd```。

```fs```任何文件操作```api```都有同步和异步两种方式，这里只演示异步```API```，同步基本也相同

### 1. 文件读写

readFile: 从指定文件中读取数据

```js
const fs = require('fs');
const path = require('path');

fs.readFile(path.resolve('aaa.txt'), 'utf-8', (err, data) => {
    console.log(err);
    console.log(data);
})

```

writeFile: 向指定文件中写入数据

```js
fs.writeFile('bbb.txt', 'hello', {
    mode: 438, // 操作位
    flag: 'w+',
    encoding: 'utf-8'
}, (err) => {
    console.log(err);
})
```

appendFile: 追加的方式向指定文件中写入数据

```js
fs.appendFile('bbb.txt', 'hello', {}, (err) => {
    console.log(err);
})
```

copyFile: 将每个文件中的数据拷贝到另一个文件

```js
fs.copyFile('aaa.txt', 'bbb.txt', (err) => {
    console.log(err);
})
```

watchFile: 对指定文件进行监控

```js
fs.watchFile('bbb.txt', {
    interval: 20 // 20ms监控一次
}, (curr, prev) => {
    console.log(curr); // 当前信息
    console.log(prev); // 前一次信息
    if (curr.mtime !== prev.mtime) {
        // 文件被修改了
    }
})

fs.unwatchFile('bbb.txt'); // 取消监控
```

### 2. 文件打开与关闭

前面我们使用了```fs```实现了文件的读写操作，既然已经读写了就证明已经实现了文件的打开，为什么```Node```还要单独的提供打开关闭的```api```呢？

因为```readFile```和```writeFile```的工作机制是将文件里的内容一次性的全部读取或者写入到内存里，而这种方式对于大体积的文件来讲显然是不合理的，因此需要一种可以实现边读编写或者边写边读的操作方式，这时就需要文件的打开、读取、写入、关闭看做是各自独立的环节，所以也就有了```open```和```close```。

```js
const fs = require('fs');
const path = require('path');

// open
fs.open(path.resolve('aaa.txt'), 'r', (err, fd) => {
    console.log(err);
    console.log(fd);

    fs.close(fd, (err) => {

    });
})
```

### 3. 目录操作

access: 判断文件或目录是否具有操作权限

```js
fs.access('aaa.txt', (err) => {
    console.log(err); // 存在错误就是没有权限
})
```

stat: 获取目录及文件信息

```js
fs.stat('aaa.txt', (err, stat) => {
    console.log(stat); // size isFile(), isDirectory()
})
```

mkdir: 创建目录

```js
fs.mkdir('a/b/c', {
    recursive: true, // 递归创建
}, (err) => {
    console.log(err);
})
```

rmdir: 删除目录

```js
fs.rmdir('a', {
    recursive: true, // 递归删除
}, (err) => {
    console.log(err);
})
```

readdir: 读取目录中内容, 不会递归子目录

```js
fs.readdir('a', (err, files) => {
    console.log(files);
})
```

unlink: 删除指定文件

```js
fs.unlink('a', (err) => {
    console.log(err);
})
```

## 6. commonjs

```CommonJS```的出现是为了解决前端模块化，他的作者希望可以倒逼浏览器们实现前端模块化，但是由于浏览器本身具备的单线程阻塞的特点，```CommonJS```并不能适用于浏览器平台。```CommonJS```是一个超集，他是语言层面的规范，模块化只是这个规范中的一个部分。

## 7. Events

```Node```中通过```EventEmitter```类实现事件统一管理。实际开发中基本很少引入这个类。这个类大部分供内置模块使用的, 比如```fs```、```http```都内置了这个模块。

```Node```是基于事件驱动的异步操作架构，内置```events```模块，模块提供了```EventEmitter```类，他的实例对象具备注册事件和发布事件删除事件的常规操作。

on：添加当事件被触发时调用的回调函数

emit: 触发事件，按照注册的顺序调用每个事件监听器

once: 注册执行一次的监听器

off：移除特定的监听器

```js
const EventEmitter = require('events');

const ev = new EventEmitter();

ev.on('event', () => {

})

ev.emit('event');
```

## 7. stream

```流```并不是```NodeJS```独创的内容，在```linux```系统中可以使用```ls | grep *.js```命令操作，其实就是将```ls```命令获取到的内容交给```grep```去处理，这就是一个流操作。

使用流可以从空间和时间上提升效率，```NodeJS```诞生之初就是为了提高```IO```性能，其中最常用的文件系统和网络他们就是流操作的应用者。

```NodeJS```中流就是处理流式数据的抽象接口，```NodeJS```中的```stream```对象提供了用于操作流的对象。对于流来说只有多使用才能加深了解。

流的分段处理可以同时操作多个数据```chunk```，同一时间流无须占据大内存空间。流配合管道，扩展程序会变得很简单。

```NodeJS```中内置了```stream```模块，它实现了流操作对象。```Stream```模块实现了四个具体的抽象。所有的流都继承自```EventEmitter```。

Readable: 可读流，能够实现数据的获取。

Writeable: 可写流，能够实现数据的写操作。

Duplex: 双工流，即可度又可写。

Tranform: 转换流，可读可写，还能实现数据转换。

```js
const fs = require('fs');

const rs = fs.createReadStream('./a.txt');
const ws = fs.createWriteStream('./b.txt');

rs.pipe(ws);
```

### 1. 可读流

生产供消费的数据流。

```js
const rs = fs.createReadStream('./a.txt');
```

```js
const { Readable } = require('stream');

const source = ['a', 'b', 'c'];

class MyReadable extends Readable {
    constructor() {
        super();
        this.source = source;
    }
    _read() {
        const data = this.source.shift() || null;
        this.push(data);
    }
}

const myreadable = new MyReadable(source);

myreadable.on('data', (chunk) => {
    console.log(chunk.toString());
})
```

### 2. 可写流

用于消费数据的流，响应。

```js
const ws = fs.createWriteStream('./b.txt');
```

```js
const { Writable } = require('stream');

class MyWriteable extends Writable {
    constructor() {
        super();
    }
    _write (chunk, en, done) {
        process.stdout.write(chunk.toString());
        process.nextTick(done);
    }
}

const mywriteable = new MyWriteable();

mywriteable.write('yindong', 'utf-8', () => {
    consoel.log('end');
})
```

### 3. Duplex

```js
const { Duplex } = require('stream');

class MyDuplex extends Duplex {
    constructor(source) {
        super();
        this.source = source;
    }
    _read() {
        const data = this.source.shift() || null;
        this.push(data);
    }
    _write(chunk, en, next) {
        process.stdout.write(chunk);
        process.nextTick(next);
    }
}

const source = ['a', 'b', 'c'];

const myDuplex = new MyDuplex(source);

mtDuplex.on('data', (chunk) => {
    console.log(chunk.toString());
})

mtDuplex.write('yindong', () => {
    console.log('end');
})

```

### 4. Transform

```js
const { Transform } = require('stream');

class MyTransform extends Transform {
    constructor() {
        super();
    }
    _transform(chunk, en, cb) {
        this.push(chunk.toString().toUpperCase());
        cb(null);
    }
}

const t = new MyTransform();

t.write('a');

t.on('data', (chunk) => {
    console.log(chunk.toString());
})
```

## 8. 链表

链表是一种数据存储结构。

在文件可写流的```write```方法工作的时候，有些被写入的内容需要在缓冲区排队等待的，而且遵循的是先进先出的规则，为了保存这些排队的数据，在新版的```Node```中就采用了链表的结构来存储这些数据。

相比较数组来说，链表的优势更明显，在多个语言下，数组存放数据的长度是有上限的，数组在执行插入或者删除操作的时候会移动其他元素的位置，并且在```JS```中数组被实现成了一个对象。所以在使用效率上会低一些。

当然这都是相对的，实际应用中数组的作用还是很强大的。

链表是由一系列节点组成的集合。这里的节点都称为```Node```节点，每个节点的身上都有一个对象的引用是指向下一个节点，将这些指向下一个节点的引用组合到一起也就形成了一个链，对于链表结构来说我们常听到的会有不同类型，双向链表，单向链表，循环链表。常用的一般是双向链表。


## 9. Assertion

断言, 如果不符合，会停止程序，并打印错误

```js
const assert = require('assert');
// assert(条件, 一段话)
function sum(a, b) {
  assert(arguments.length === 2,  '必须有两个参数');
  assert(typeof a === 'number',  ‘第一个参数必须是数字’);
}
```

## 10. C++ Addons

使用c语言写的插件，在Node中使用。

## 11. Cluster

多线程

一个程序就是一个进程，一个进程会有多个线程
进程和进程是严格隔离的，拥有独立的执行空间。
同一个进程内的所有线程共享一套空间、代码

1. 多进程

成本高，速度慢，安全，进程间通信麻烦，写代码简单。

2. 多线程

成本低，速度快，不安全，线程间通信简单，写代码复杂。

```Child_Processes```, ```Cluster```, ```Process```。

## 12. Command Line Options

获取命令行参数

## 13. Crypto

签名，完成加密算法的

```js
const cryptp = require("crypto");
let obj = cryto.createHash('md5');
obj.update('123');
console.log(obj.digest('hex')); // MD5 32位
```

## 14. OS

系统相关

```js
const os = require('os');
// 获取cpu个数
os.cpus(); 
```

## 15. Events 

事件队列

```js
const Event = require('events').EventEmitter;
let ev = new Event();
// 绑定事件
ev.on('msg', function(a, b,c) {
    console.log(a, b, c); // 12 ， 3， 8
})
// 派发事件
ev.emit('msg', 12,3,8);
```

自己实现一个```Events```:

```js
class EventEmitter {
    constructor() {
        this._events = {};
    }
    on(eventName, callBack) {
        if (this._events[eventName]) {
            this._events[eventName].push(callBack);
        } else {
            this._events[eventName] = [callBack];
        }
        
    }
    emit(eventName) {
        this._events[eventName].forEach(fn => {
            fn();
        })
    },
    off(eventName, callBack) {
        this._events[eventName] = this._events[eventName].filter(fn => fn !== callBack)
    }
}

```

## 16. url

请求```url```模块

```js
const url = require('url');
url.parse('url', true); // 会解析出url和参数，包含query-string的功能。
```

## 17. Net

```TCP``` 稳定  ```Net```

```UDP``` 快 ```UDP/Datagram```

```DNS/Domain```

## 18. DNS

域名解析成```ip```

```js
const dns = require('dns');
dns.resolve('baidu.com', (err, res) => {
})
```

## 19. http

基于```net```的```http```服务

```js
const http = require('http');
const server = http.createServer((request, response) => {
    // request 为请求数据
    // response 为响应数据
    // 设置返回值类型
    res.writeHead(200, {'Content-type' : 'text/html'});
    response.write = 'server start';
    return response.end(); // 结束
});
server.listen(3000); // 服务监听的端口号
```