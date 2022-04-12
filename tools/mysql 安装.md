## 1. 概述

```CentOS 8```操作系统默认已经开启```MySQL 8.0```，可从默认的```CentOS 8```存储库中安装最新版本的```MySQL```数据库服务器```8.0```版(首先先确保我们一定要有网的情况下安装)

## 2. 安装

通过以```root```用户使用```CentOS```软件包管理器来安装```MySQL 8.0```服务器：

```s
sudo dnf install @mysql
```

下载完毕之后

直接启动```mysql```服务

```s
systemctl start mysqld.service
```

## 3. 设置密码

启动完成之后直接打开```mysql```，用```root```用户登入即可，由于并没有设置密码。所以填写密码的环节直接回车即可登录。

```s
mysql -u root -p
```

登入成功之后可以修改密码，注意密码比较复杂，需要字母数字混合，简单密码会修改失败。

```s
ALTER USER 'root'@'localhost' IDENTIFIED BY 'your_password';
```

## 4. 关闭防火墙

关闭防火墙或者开启我们的```3306```端口就可以进行访问了

```s
# 查看firewall服务状态
systemctl status firewalld

# 开启、重启、关闭、firewalld.service服务
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop

# 查看防火墙规则
firewall-cmd --list-all    # 查看全部信息
firewall-cmd --list-ports  # 只看端口信息

# 开启端口
开端口命令：firewall-cmd --zone=public --add-port=80/tcp --permanent
重启防火墙：systemctl restart firewalld.service

命令含义：
--zone #作用域
--add-port=80/tcp  #添加端口，格式为：端口/通讯协议
--permanent   #永久生效，没有此参数重启后失效
# 关闭防火墙(不建议)
systemctl stop firewalld.service
# 开启端口（建议）
firewall-cmd --zone=public --add-port=3306/tcp --permanent
# 成功之后我们需要重启我们的防火墙
systemctl restart firewalld.service
```

## 5. 赋予权限

```s
# 创建本地用户
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';

# 新建远程用户
CREATE USER 'user'@'%' IDENTIFIED BY 'password';

# 新建数据库
CREATE DATABASE test_db;

# 查看用户权限
SHOW GRANTS FOR 'user'@'%';

# 赋予用户指定数据库远程访问权限
GRANT ALL PRIVILEGES ON test_db.* TO 'user'@'%';

# 赋予用户对所有数据库远程访问权限
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';

# 赋予用户对所有数据库本地访问权限
GRANT ALL PRIVILEGES ON *.* TO 'user'@'localhost';

# 刷新权限
FLUSH PRIVILEGES;
```

## 6. 收回权限

```s
# 收回权限
REVOKE ALL PRIVILEGES ON *.* FROM 'test'@'%';

# 删除本地用户
DROP USER 'user'@'localhost';

# 删除远程用户
DROP USER 'user'@'%';

# 刷新权限
FLUSH PRIVILEGES;
```

## 7. 远程登录

在```mysql```数据库查看```user```表信息 ：

```s
use mysql;
select host， user， authentication_string， plugin from user;
```

表格中```root```用户的```host```默认是```localhost```，只允许本地访问。授权```root```用户的所有权限并设置远程访问：
```s
# 授权
GRANT ALL ON *.* TO 'root'@'%';

# 刷新
FLUSH PRIVILEGES;
```

```root```用户默认的密码加密方式是：```caching_sha2_password```；而很多图形客户端工具可能还不支持这种加密认证方式，连接的时候就会报错 。通过以下命令重新修改密码：

```s
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'your password';
```
这里指定了```root```的密码加密方式为```mysql_native_password```，如果想改变默认密码加密方式都是，可以在```/etc/my.cnf```文件加上一行：

```s
default-authentication-plugin=mysql_native_password
```
如果服务器开启了防火墙，则需要打开```3306```端口。

```s
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload
```
注意：如果是云服务器，有的服务商（如阿里云）需要到控制台去开放端口的。

## 8. 修改字符编码

字符集是一套符号和编码，查看字符集配置：

```s
mysql> show variables like 'charac%';
+--------------------------+--------------------------------+
| Variable_name            | Value                          |
+--------------------------+--------------------------------+
| character_set_client     | utf8mb4                        |
| character_set_connection | utf8mb4                        |
| character_set_database   | utf8mb4                        |
| character_set_filesystem | binary                         |
| character_set_results    | utf8mb4                        |
| character_set_server     | utf8mb4                        |
| character_set_system     | utf8                           |
| character_sets_dir       | /usr/share/mysql-8.0/charsets/ |
+--------------------------+--------------------------------+
```

字符集生效规则为：```Table```继承于```Database```，```Database```继承于```Server```。就是说，可只设置```character_set_server```。

校对规则是在字符集内用于比较字符的一套规则，查看校对规则：

```s
mysql> show character set like 'utf8%';
+---------+---------------+--------------------+--------+
| Charset | Description   | Default collation  | Maxlen |
+---------+---------------+--------------------+--------+
| utf8    | UTF-8 Unicode | utf8_general_ci    |      3 |
| utf8mb4 | UTF-8 Unicode | utf8mb4_0900_ai_ci |      4 |
+---------+---------------+--------------------+--------+
```

校对规则生效规则：如果没有设置校对规则，字符集取默认校对规则，例如```utf8mb4```的校对规则是```utf8mb4_0900_ai_ci```。

```MySQL 8```默认字符集改成了```utf8mb4```。之前的```MySQL```版本如果默认字符集不是```utf8mb4```，建议改成```utf8mb4```。

```mb4```即```most bytes 4```。为什么是```utf8mb4```，而不是```utf8```，```MySQL```支持的```utf8```编码最大字符长度为```3```字节，如果遇到```4```字节的宽字符就会插入异常。

下面是 老版```MySQL```修改字符集为```utf8mb4```的步骤，```MySQL 8.0+```无需修改。

```s
# 查看配置文件位置
whereis my.cnf

# 打开文件
vi /etc/my.cnf
```

增加字符编码配置项：

```s
[client]
default-character-set=utf8mb4

[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci
```

重启```MySQL```服务

```s
sudo systemctl restart mysqld
```

使用```MySQL```命令检查字符集配置：

```s
show variables like 'charac%';
```

## 9. 设置软连接

```s
ln -s  /usr/local/mysql/bin/mysql    /usr/bin
```

## 10. 卸载mysql

```s
# 查找已经安装的mysql
rpm -qa | grep -i mysql
# 删除对应的mysql
yum -y remove mysql-*
# 查找一些目录
find / -name mysql
# 删除找到的目录
rm -rf /var/lib/selinux/targeted/active/modules/100/mysql
# 删除配置文件，主要mysql的默认密码，如果不删除，以后安装mysql这个sercret中的默认密码不会变，使用其中的默认密码就可能会报类似Access denied for user 'root@localhost' (using password:yes)的错误.
rm -rf /root/.mysql_sercret
```
