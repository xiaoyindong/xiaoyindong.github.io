## 1. 概述

```Jenkins```是一个```Java```编写的开源的持续集成工具。支持主流软件配置管理，配合实现软件配置管理，持续集成功能。

开发人员很多的工作就是在命令与脚本间游走，这是与系统交互最为紧密的通信工具，但缺点也很明显，可以简单形容为一个黑盒。

```Jenkins```的出现彻底打开了这个黑盒，让开发人员和运维人员协同工作在一个白盒之中，他在我们运维工作之中起到的是一个承上启下的作用。

前台页面可以直接收集到所有```job```的相关信息，而且它能够作为一个```pipline```将开发周期中各个环节利用```stage```组合到一起。无论是哪种工具，都能串联成一个框架，形成一个高效的可扩展的平台。

```Jenkins```是主流的运维开发平台，兼容所有主流的开发环境，插件市场可与海量业内主流开发工具实现集成。```Job```为配置单位与日志管理，使运维与开发人员能协同工作。

权限管理划分不同```Job```不同角色，强大的负载均衡功能，保证项目的可靠性。

## 2. 安装

### 1. 设置镜像源

```s
# 安装wget
yum install wget -y
# 设置镜像源
wget -o /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat-stable/jenkins.repo
# 导入yum的key验证安全性
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

### 3. 安装java

```jenkins```依赖```8.0```以上的```java```环境。

```s
yum -y install java

java -version
```

### 4. 关闭防火墙

```s
systemctl stop firewalld
# 禁止开机启动
systemctl disable firewalld
```

### 5. 关闭强制访问安全策略。

```s
vi /etc/sysconfig/selinux

SELINUX=disabled # enforcing -> disabled
# 重启
reboot
# 查看是否禁用成功
getenforce
```

### 6. 安装```jenkins```

```s
yum install jenkins -y
```

### 7. 创建```deploy```用户作为```jenkins```系统服务用户

```s
useradd deploy

vi /etc/sysconfig/jenkins
```

### 8. 修改配置

修改```JENKINS_USER```用户为```deploy```， 修改```JENKINS_PORT```端口为```8080```。

```s
JENKINS_USER="deploy"
...
JENKINS_PORT="8080"
```

### 9. 修改```lib```和```日志```的权限。

```s
chown -R deploy:deploy /var/lib/jenkins
chown -R deploy:deploy /var/log/jenkins
# 启动jenkins
systemctl start jenkins
# 确认是否启动
lsof -i:8080
```
### 10. 安装完成

可以设置一条```jenkins```的```dns```配置，方便使用域名访问```jenkins```

```s
xx.xx.xx.xx jenkins.example.com
```

在浏览器中访问```jenkins.example.com:8080```。

首次访问会要求我们输入管理员密码，在日志```/var/lib/jenkins/secrets/initialAdminPassword```中会找到管理员的初始密码。

登录后会进入```jenkins```的插件安装界面，选择后会自动安装到本地。

安装完成之后会要求创建第一个管理员用户。

## 3. 界面

管理界面的布局由若干个区域构成，左边是```菜单列表```。

新建任务可以创建一个```job任务```；用户菜单可以创建用户；构建历史可以查看构建```log日志```；系统管理是```系统配置```选项；```我的视图```和```新建视图```，可以自定义管理界面的布局；凭据可以添加```git```的密码，```ssh```的公钥等，作为```jenkins```的凭据。

构建队列就是所有构建任务的队列，当有多个任务的时候都会在列表中显示。

系统管理中的系统设置用来定义```jenkins```的全局配置，全局安全配置用来设置用户是否可以注册等，可以配置每一个用户对```jenkins```任务的操作权限。管理插件就是插件市场平台，可以安装和更新插件。
