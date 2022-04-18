## 1. 应用

在常规的前端项目中，部署项目需要经过本地```build```，压缩文件，将压缩包上传至服务器并解压文件等步骤，过程较为繁琐。本文编写一个```nodejs```脚本，用来告别手动上传的过程，配置使用简单，实现前端一键自动化部署。

## 2. 依赖工具

```archiver```用于压缩文件

```node-ssh```通过```ssh```链接服务器

## 3. 安装

1. 安装archiver node-ssh 依赖

```js
npm install archiver node-ssh --save-dev
```

## 4 编写脚本

在项目跟目录新建```deploy.js```

```js
const path = require('path');
const archiver =require('archiver');
const fs = require('fs');
const node_ssh = require('node-ssh');
const ssh = new node_ssh();

// 上传服务器代码
const upload = () => {
    // 链接远程服务器
    ssh.connect({
       host: '192.xxx.x.xxx',
       username: 'root',
       password: 'xxxx',
       port:22
   }).then(function () {
       // 上传网站的发布包至服务器中的位置，(__dirname + '/dist.zip' 为本地文件位置， '/home/dist.zip'：服务器中的位置)
       ssh.putFile(__dirname + '/dist.zip', '/home/dist.zip').then(status => {
               console.log('上传文件成功');
               console.log('开始执行远端脚本');
               // 上传成功后触发远端脚本，此处执行服务器脚本，代码参考步骤4
         }).catch(err=>{
            console.log('文件传输异常:',err);
            process.exit(0);
         });
   }).catch(err=>{
       console.log('ssh连接失败:',err);
       process.exit(0);
   });
}

// 将dist文件夹压缩成zip ---- start -----
// 设置初始化压缩参数
const archive = archiver('zip', { zlib: { level: 9 }}); 

// 创建压缩后的文件
const output = fs.createWriteStream(__dirname + '/dist.zip');
// 监听压缩事件
output.on('close', (err) => {
    if (err) {
        console.log('关闭archiver异常:', err);
        return;
    }
    // 压缩完成
    console.log(`----压缩文件总共 ${archive.pointer()} 字节----`);
    // 调用上传方法
    upload();
});

// 通过管道方法设置输出文件的位置
archive.pipe(output);//典型的node流用法
// 打包dist里面的所有文件和目录
archive.directory('./dist');
// 完成
archive.finalize();
```

## 5. 创建服务器对应脚本

```deploy.sh```

```s
#!/bin/bash

# 进入文件所在目录
cd /home

# 删除原静态资源目录
rm -rf dev

# 进入文件所在目录
cd /home

#解压新的包
unzip dist.zip

# 将解压出的dist目录修改名称为dev
mv dist dev

```

## 6. 文件上传后，执行服务器脚本

```js
ssh.execCommand('sh deploy.sh', { cwd: 'deploy.sh所在绝对路径' }).then(result => {
    console.log('远程STDOUT输出: ' + result.stdout)
    console.log('远程STDERR输出: ' + result.stderr)
    if (!result.stderr) {
        console.log('发布成功!');
        process.exit(0);
    }
});
```
