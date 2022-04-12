## 1. 概述

搭建的服务器至少内存要是```4G```,因为```gitlab```比较吃内存, 可以查看gitlab[官网](https://about.gitlab.com/install/)。

```gitlab```是一个开源分布式版本控制系统，他的开发语言是```ruby```。通过```UI```界面与用户交互实现项目代码管理，版本控制，代码复用与查找。

## 2. 安装

1. 首先需要关闭CentOS7系统下的放火墙。

```s
systemctl stop firewalld
# 禁止开机启动
systemctl disable firewalld
```

2. 关闭强制访问安全策略。

```s
vi /etc/sysconfig/selinux

SELINUX=disabled # enforcing -> disabled
# 重启
reboot
```

3. 稍等片刻重新链接主机

```s
# 查看是否禁用成功
getenforce
```

4. 安装gitlab的依赖文件

```s
yum install curl policycoreutils openssh-server openssh-clients postfixs
```

5. 下载gitlab仓库源。

```s
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

6. 启动postfix邮件服务

```s
systemctl start postfix
systemctl enable postfix
```

7. 安装gitlab-ce社区版本服务包, 启动安装向导。

```s
yum -y install gitlab-ce
```

7. 安装gitlab-ce社区版本服务包, 启动安装向导。

```external_url```是访问域名，可以不设置，后面再添加

```s
# 带域名安装
external_url="http://gitlab.example.com" yum -y install gitlab-ce
# 不带域名安装
yum -y install gitlab-ce
```

8. 初始化配置

```s
gitlab-ctl reconfigure
```

9. 查看状态

安装完成直接运行下面命令查看状态。

```s
# 查看gitlab状态
gitlab-ctl status
```
然后可以使用```ip```或者设置的域名在浏览器访问这个地址。

9. 设置密码

旧版的```gitlab```首次访问会进入设置密码页面，后面就可以使用设置的密码登录管理账号。

新版本可以进入```radis```日志修改密码。

```s
# 进入radis日志
gitlab-rails console

# 首先查询管理员账号id。
user = User.where(username:'root').first
#<User id:1 @root>

# 设置密码为123456789
user.password = '123456789'

# 确认密码
user.password_confirmation = '123456789'

# 保存
user.save!

# Enqueued ActionMailer::MailDeliveryJob (Job ID: 5541291d-d7e5-4684-b92d-bc5386a6aea5) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", {:args=>[#<GlobalID:0x00007f539eacbdd8 @uri=#<URI::GID gid://gitlab/User/1>>]}
# => true

# 退出
quit
```

```save```如果报错的话，检查下密码的长度，需要```8```位以上

接着可以使用用户名```root```及密码```123456789```登录了。

除了上面方式还可以```id=1```定位超级管理员，修改密码。

```s
u= User.where(id: 1).first

u.password='123456789'
# => "new_password" 
# 保存用户密码
u.save!

# Enqueued ActionMailer::DeliveryJob (Job ID: 99118288-b58b-4d52-94c1-28979bcb63e8) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", gid://gitlab/User/1
# => true

# 退出
quit
```

10. 登录

可以在浏览器中使用```ip```或者域名访问。

## 3. gitlab应用

```gitlab```的后台管理可以针对不用的项目不同用户去定制不同的访问策略，开发与运维的两个角色可以各司其职互不影响的在自己所在的场景下工作。

作为开发人员关注的点肯定是代码的快速发布和审核，一般项目测试之后我们会提交一个```master```分支合并的申请等待领导去审核，决定是否确认合并操作，确认之后开发人员会在另一个```fueture```分支继续工作。

作为运维人员关注的另一个点是保证```gitlab```的维护和管理，例如```CPU```利用率，内存使用情况。

运维人员点击```设置``` -> ```Monitoring``` -> ```System Info```这里面包含了若干系统资源的状态值。

最后，如果你是本地测试而不像校验https时，可以使用```http.sslVerify=false```禁用。
```s
git -c http.sslVerify=false clone http://gitlab.example.com/test/test1.git
```

## 4. 修改域名

如果要是设置域名，可以在运行```yum install -y gitlab-ce```命令的时候通过```EXTERNAL_URL```设置。这个域名可以是真实购买的域名，如果你要把```gitlab```安装到公网比如阿里云上的话。

```s
external_url="http://gitlab.example.com" yum install -y gitlab-ce
```

也可以安装之后通过修改```/etc/gitlab/gitlab.rb```修改域名。

```s
#查看当前绑定的域名或者IP
grep "^external_url" /etc/gitlab/gitlab.rb

#打开配置文件
vi /etc/gitlab/gitlab.rb

# external_url 'http://xx.xx.xx.xx'  #替换   #修改成域名访问
external_url 'http://gitlab.new.com'  
```

修改之后要 ```gitlab-ctl reconfigure``` 重新跑一下配置，否则不生效。

```s
gitlab-ctl reconfigure
```

如果机器```80```端口被占用，需要修改```gitlab```端口

```s
#查看默认端口
grep "'listen_port" /etc/gitlab/gitlab.rb

 #打开配置文件
vi /etc/gitlab/gitlab.rb

#找到取消注释，修改端口为8080
nginx['listen_port'] = 8080

#重新跑一下配置
gitlab-ctl reconfigure
```

由于更改端口，域名解析不到。提供方式使用云服务中的负载均衡。记得修改安全组。

还要修改修改```nginx```中```gitlab```配置文件```vi ~git/nginx/conf/gitlab-http.conf```

```s
server {
  listen *:8080; 保持端口一致
  server_name http://gitlab.example.com;
    ...
}
```

重启```gitlab```

```s
gitlab-ctl restart
```

## 5. 配置https

使用```openssl```命令创建```gitlab```本地证书并配置```config```加载该证书。

```s
#创建ssl命令
mkdir -p /etc/gitlab/ssl

# 创建本地私有秘钥
openssl genrsa -out "/etc/gitlab/ssl/gitlab.example.com.key" 2048
```

使用```openssl```生成```csr```证书, 这里```country```可以输入```cn```，```province```可以输入```hz```，```city```输入```hz```，```organization```输入```空格```，```unit name```也输入```空格```，```common name```输入的是```gitlab的域名gitlab.example.com```, 邮箱地址输入```admin@example.com```, 证书密码输入```123456```，```company name```直接```回车```就可以。

```s
openssl req -new -key "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.csr"

# 创建签署证书
openssl x509 -req -days 3650 -in "/etc/gitlab/ssl/gitlab.example.com.csr" -signkey "/etc/gitlab/ssl/gitlab.example.com.key" -out "/etc/gitlab/ssl/gitlab.example.com.crt"

# 使用openssl生成pem证书
openssl dhparam -out /etc/gitlab/ssl/dhparams.pem 2048

# 查看证书 正常应该有4个
cd /etc/gitlab/ssl/
ll

# 修改所有证书的权限为600
#进入ssl文件夹
cd /etc/gitlab/ssl/
#更改本文件夹下所有文件的权限
chmod 600 *

# 编辑gitlab.rb，将所有的证书配置到gitlab的配置文件中
vi /etc/gitlab/gitlab.rb
```

找到```external_url```这行，将```http```改为```https```

```s
https://gitlab.example.com
```

找到nginx['redirect_http_to_https']，移除```#```并将值改为```true```;

找到nginx['ssl_certificate']，值修改为```/etc/gitlab/ssl/gitlab.example.com.crt```;

找到nginx['ssl_certificate_key']，值修改为```/etc/gitlab/ssl/gitlab.example.com.key```;

找到nginx['ssl_dhparam']，值修改为```/etc/gitlab/ssl/dhparams.pem```;

修改完这些之后我们需要初始化```gitlab```配置

```s
#如果不执行此命令，此时可能会卡住报错
systemctl restart gitlab-runsvdir
#初始化
gitlab-ctl reconfigure
```

当出现```gitlab Reconfigured```就是正常初始化结束。

最后需要修改```gitlab```的```nginx```的代理文件，配置```https```。

```s
vi /var/opt/gitlab/nginx/conf/gitlab-http.conf
```

找到```server```，在```server_name```下增加：```rewrite ^(.*)$ https://$host$1 permanent```;

重启```gitlab```使```Nginx```配置生效。

```s
gitlab-ctl restart
```

## 6. 使用镜像安装

如果是国内的话，可以尝试使用清华大学的源，这样快些。

官方源地址：```https://about.gitlab.com/downloads/#centos7```

清华大学镜像源：```https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce```


```s
cat> /etc/yum.repos.d/gitlab-ce.repo<< EOF
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/  
gpgcheck=0
enabled=1
EOF
```

如果在国外的话可以使用

```s
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```