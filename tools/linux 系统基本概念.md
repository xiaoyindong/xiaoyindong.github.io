## 1. 概述

```Linux```是一款免费使用和自由传输的类Unix操作系统，在服务器领域和嵌入式领域有非常广泛的应用。```Linux```分为内核版和发行版本，各个厂商会制作自己的发行版本```redhat```、```CentOS```、```debian```、```ubuntu```、```fedora```。

```Linux```严格区分大小写，所有的内容以文件形式保存，比如用户和文件，不靠扩展名区分文件类型，靠权限来区分，但是有一些约定的扩展名，是给管理员看的。

| 类别 | 扩展名 |
| ---- | ---- |
| 压缩包 | .gz  .bz2 .tar  .tgz |
| 二进制文件 | .rpm |
| 网页文件 | .html .php |
| 脚本文件 | .sh |
| 配置文件 | .conf |

```windows```的程序不能直接在```Linux```中安装和运行，```Linux```更多使用字符界面，占用系统资源更少，减少了出错和被攻击的可能性，会让系统更稳定。

## 2. 计算机如何启动

```BIOS```计算机通电后，第一件事就是读取刷入```ROM```芯片的开机程序，这个程序叫做基本输入系统```Basic``` ```Input.Outoyr``` ```System```，```BIOS```程序首先检查硬件能否满足运行的基本条件，这叫硬件自检，如果硬件出现问题，会蜂鸣报警，如果没问题，显示硬件信息。启动顺序是主引导记录到分区表...。

## 3. x-shell

是一块远程连接工具，```4.0```版本是免费的，```5.0```以上版本开始收费。

## 4. Linux目录说明

| 目录 | 备注 |
| --- | --- |
| /boot | 启动目录，启动相关文件 |
| /dev | 设备文件 |
| /etc | 配置文件 |
| /home | 普通用户的家目录，可以操作 |
| /lib | 系统库保存目录 |
| /mnt | 移动设备挂载目录 |
| /media | 光盘挂载目录 |
| /misc | 磁带机挂载目录 |
| /root | 超级用户的家目录 可以操作 |
| /tmp | 临时目录 可以操作 |
| /proc | 正在运行的内核信息映射 主要输出进程信息，内存资源信息和磁盘分区信息等等 |
| /var | 变量 |
| /sys | 硬件设备的驱动程序信息 |
| /bin | 普通的基本命令，如is，chrmod等，一般的用户也可以使用 |
| /sbin | 基本的系统命令，如shutdown，reboot 用于启动系统，修复系统，只有管理员可以运行 |
| /usr/bin | 是你在后期安装的一些软件的运行脚本 |
|/usr/sbin  | 放置一些用户安装的系统管理的必备程序 |
| ~ | 家位置，每个人位置不一样，用户在home下，管理员在root下 |
| [root@localhost~] # | 当前用户名，主机名，工作目录 |

## 5. 命令提示符

| 命令 | 备注 |
| --- | --- |
| setup | 设置 |
| service network restart | 重启网络 |
| cd / | 根目录 |
| ls -a | 显示所有文件，包括隐藏文件 |
| ls -al |详细信息  |
| mkdir | 新建文件夹 |
| ls -ld ·文件夹名称 | 查看文件夹详细信息 |
| mkdir -p a/b/p/aa | 级联创建目录 |
| cd change directory | 切换目录 |
| pwd | 查看当前目录 |
| rmdir `文件夹名` | 删除目录 |
| rm -rf `文件夹名` | 强力删除目录，递归删除 remove re force |
| rm -rf / | 删除根目录所有文件 |
| cp `文件1` `文件夹2` | 把文件1 拷贝 到文件夹2中 |
| cp -r `文件夹1` `文件夹2` | 把文件夹1 拷贝到文件夹2，整个拷贝过去 |
| mv `旧文件名` `新文件名` | 同目录重命名 不同目录剪切 |

## 6. ln

链接文件

硬链接是拥有相同的节点和存储```block```块，可以看做是同一个文件，可以通过节点访问，但不能跨分区也不能针对目录使用，只能针对文件。一般并不使用。

```s
touch a.txt // 新建文件
vi a.txt // 编辑文件
cat a.txt // 查看文件内容
ln a.txt aa.txt; // 建立链接，复制一个aa.txt copy a.txt
```
软连接通过```ln -s 原文件 目标文件```，类似```windows```快捷方式，软连接拥有自己的节点和```block```块，但是数据块中只保存原文件的文件名和节点号，并没有实际的文件数据。软连接的文件权限都是```777```，```lnwxrwxrwx! 软连接```。

修改任意一个文件，另一个也会改变，删除了原文件，软连接就不能使用了，并且软连接原文件必须写绝对路径。

## 7. 新建用户

```useradd user1```

## 8. 文件搜索

在后台数据库中按文件名搜索，速度比较快，数据保存在```/var/lib/miocate``` 后台数据库，每天更新一次，可以用```updatedb```命令立刻更新数据库。

只能通过文件名进行搜索```/etc/updatedb.conf```建立索引的配置文件。

```s
PRUNE_BLIND_MOUNTS="yes" # 全部生效 开启搜索限制
PRUNEFS # 不搜索的文件系统
PRUNENAMES # 忽略的文件类型
PRUNEPATHS # 忽略的路径 /tmp
```

```whereis```查询命令所在的位置，及帮助文件。

```s
whereis ls 
```

```which```可以看到别名，能看到的都是外部安装的命令，无法查看```shell```自带的命令。

```alias```设置别名。

```s
alias ls="ls -l --color=auto"
```
### 1. find

```s
find . -name a.txt # 通过名称查找
find . -name a.t* # 模糊匹配
find . -name a.t[xyz]t # 枚举值
find . -iname a.tXT # 忽略大小写
find . -user root # 通过所有者
find . -user root # 通过所有者

find . -size 10k # 等于10k
find . -size +10k # 大于10k
find . -size -10k # 小于10k
```
### 2. 管道符

```s
cat a.txt | grep one # 过滤包含one的那一行, 查找出包含one的那一行
cat a.txt | grep -v one # 除了one的
cat a.txt | grep -i one # 不区分大小写
```

## 9. 关机和重启

```s
shutdown -r 22:00 # 22点重启
shutdown -h 22:00 # 22点关机
shutdown -c # 取消前一个关机
```

## 10. w

查看当前的登录用户