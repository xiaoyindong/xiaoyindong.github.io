## 1. 概述

```ansible```是一个基于```Python```的开源部署工具，依赖```ssh```实现模块化部署管理。兼容```windows```和```linux```平台。推送```Playbook```进行远程节点快速部署，易于上手，适合中小规模快速部署。

```ansible```与```Chef```，```Saltstack```不同。```Chef```是```Ruby```语言编写，```C/S```架构，配置需要依赖```GIT```，```Recipy```脚本编写规范需要一定的编程经验。

```Saltstack```也是```Python```语言编写的```CS```架构，模块化配置管理，```YAML```脚本编写规范，适合大规模集群部署。他的优点是内部含有一个异步服务器，可以为客户端加快文件服务速度。

```ansible```轻量级无客户端，有客户端就会要求服务器提供链接端口，无形中就会占用内存和```CPU```，而且如果这台机器安装了```java```服务，那么提供的这个端口就可能存在安全隐患。

```ansible```开源免费，学习成本低，可以快速上手。他将```YAML```脚本命名为```Playbook```，这样就定义了一个统一的编写框架，可以遵循官方的说明与语法文档对产品逻辑进行编写。

```ansible```完善的模块化扩展功能，支持目前主流的开发场景。具有强大的稳定性和兼容性，得益于```Linux```默认的```Python```和```ssh```。

## 2. 安装

```ansible```是```Python```开发的，所以需要先安装```Python```。首先要保证这里安装的```Python```只会提供给```ansible```使用，而不会给其他工具使用，这样保证安全问题。

推荐使用```pythonenv```去隔离```python3.6```环境，可以单独划分一个```python```给```ansible2.5```使用。

### 1. 关闭防火墙

```s
systemctl stop firewalld
# 禁止开机启动
systemctl disable firewalld
```

### 2. 关闭强制访问安全策略。

```s
vi /etc/sysconfig/selinux

SELINUX=disabled # enforcing -> disabled
# 重启
reboot
# 查看是否禁用成功
getenforce
```

### 3. 下载并安装python

需要保证系统已经安装了```gcc```和```zlib```，没有的话先安装。

```s
# 安装gcc
yum install -y gcc
# 安装zlib
yum -y install zlib*

wget http://www.python.org/ftp/python/3.6.5/Python-3.6.5.tar.xz

tar xf Python-3.6.5.tar.xz

cd Python-3.6.5

./configure --prefix=/usr/local/ --with-ensurepip=install --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"

make && make altinstall
```

### 4. 查看pip3.6路径

```s
which pip3.6
# 设置软连接
ln -s /usr/local/bin/pip3.6 /usr/local/bin/pip
```

### 5. 安装virtualenv工具

```s
pip install virtualenv -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com
```

### 6. 创建ansible系统账户

安装```Py3.6```版本的```virtualenv```实例。

```s
useradd deploy

su - deploy

virtualenv -p /usr/local/bin/python3.6 .py3-a2.5-env
```

### 7. git源码安装ansible2.5版本。

```s
cd /home/deploy/.py3-a2.5-env
# 查看git，如果没有安装git需要安装git su - root    yum -y install git nss curl
which git
# git安装之后切换回deploy
su - deploy
```

### 8. clone源代码到本地

```s
git clone https://github.com/ansible/ansible.git
# 加载py3.6环境
source /home/deploy/.py3-a2.5-env/bin/activate
# 安装
pip install paramiko PyYAML jinja2
```

### 9. 将下载的源代码移动到虚拟环境下。

```s
mv ansible .py3-a2.5-env/

cd .py3-a2.5-env/ansible/

git checkout stable-2.5

source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q
```

### 10. 验证是否加载安装完成

```s
ansible --version
```
