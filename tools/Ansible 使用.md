
## 1. Playbooks入门和编写规范

```Playbooks```是```ansiable```的编排语言框架，它本身的简单易读的语法结构以及丰富的内建模块非常方便编写远程部署策略。

```Playbooks```基础的文件格式为```yml```文件格式。```inventory```存放```server```详细清单目录，```roles```保存要部署的详细任务列表。```testbox```表示详细任务, ```main```住任务文件。```deploy```任务入口文件。

```yml
inventory/
    testenv
roles/
    testbox/
        tasks/
            main.yml
deploy.yml
```

```testenv```由上下两部分组成，```testservers```表示```server```组列表，也就是要部署的服务器的列表，值为目标部署服务器的主机名。```testservers```表示目标主机使用的参数列表。

```yml
[testservers]
test.example.com

[testservers:vars]
server_name=test.example.com
user=root
output=/root/test.txt
```

上面的意思是给```test.example.com```这个主机定义了```server_name```, ```user```以及```output```参数。

```main.yml```主任务文件用来保存特性```roles```下面具体执行的任务。这里会保存一个或多个```task```作为任务，```task```一般由两部分组成，```任务名称```和```执行的脚本```，脚本通常调用内建模块编排执行逻辑。

```yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
```

这里打印的```user```，```server_name```和```output```就是前面```testenv```定义的值。

入口文件```deploy.yml```，他直接和```ansible```对话。这里的```host```对应```testenv```中的```testservers```。

```gather_facts```获取```Server```基本信息，```remote_user```指定目标服务器系统用户，```roles```是进入```roles/testbox```任务目录进行执行。

```yml
- hosts: "testservers"
  gather_facts: true
  remote_user: root
  roles:
    - testbox
```

因为```ansible```是使用```ssh```作为通信协议，为了保证正常的通信，需要配置```ansible```主机与目标主机的秘钥认证，保证无需密码即可访问目标主机。

```ansible```服务端创建```ssh```本地秘钥

```s
ssh-keygen -t rsa
```

```ansible```服务器端建立与目标部署机器的秘钥认证。将```ansible```服务器的公钥传输到目标服务器中，可以实现直接连接。

```s
ssh-copy-id -i /home/deploy/.ssh/id_rsa.pub root@test.example.com
```

部署到```testenv```环境

```s
ansible-playbook -i inventory/testenv ./deploy.yml
```

实时演示一下。 需要创建一个```testbox```虚拟机主机。同样安装```CentOS7```。尝试通过```ansible```主机将文件远程传输至```testbox```虚拟主机。

首先```ssh```登录到```ansible```主机上，然后切换到```ansible2.5```版本。

```s
su - deploy 
source .py3-a2.5-env/bin/activate
source .py3-a2.5-env/ansible/hacking/env-setup -q
ansible-playbook --version
```

编写```playbooks```框架

```s
mkdir test_playbooks
cd test_playbooks
mkdir inventory
mkdir roles
cd inventory/
vi testenv
```

编辑```testenv```

```s
[testservers]
test.example.com

[testservers:vars]
server_name=test.example.com
user=root
output=/root/test.txt
```

创建```main.yml```

```s
cd ..
cd roles/
mkdir testbox
cd testbox
mkdir tasks
cd tesks/
vi main.yml
```

编辑```main.yml```, 添加一个测试任务，这里```shell```前是两个空格。

```yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
```

返回路径，回到```test_palybooks```

```s
cd ../../..
```

创建```deploy.yml```文件

```s
vi deploy.yml
```

编辑这个文件, 告诉```ansible```使用```host```任务，并且在目标主机中获取主机信息，并且使用目标主机的```root```账户读写权限。最后告诉```ansible```执行```roles```下的```testbox```任务。

```yml
- hosts: "testservers"
  gather_facts: true
  remote_user: root
  roles:
    - testbox
```

使用```tree```查看一下目录结构

```s
tree .
```

这样文件就配置好了，接下来要配置一下秘钥认证

```s
# 返回root命令行
su - root
# 编辑dns
vi /etc/hosts
```

添加dns文件。

```s
xx.xx.xx.xx test.example.com
```

返回到```deploy```命令行

```s
exit
```

创建```ssh```秘钥认证对, 一路回车。

```s
ssh-keygen -t rsa
# cat ~/.ssh/id_*.pub | ssh  root@test.example.com 'cat >> .ssh/authorized_keys'
ssh-copy-id-i ~/.ssh/id_rsa.pub root@test.example.com
```

这样就可以不需要密码直接连接到```test.example.com```服务器了，这里在```ansible```的```deploy```目录中，执行入口文件。

```s
cd test_playbooks/

ansible-playbook -i inventory/testenv ./deploy.yml
```

执行完成。可以去目标主机查看状态。创建了一个```test.txt```文件并且将数据写到了文件中。

```s
ssh root@test.example.com
ls
cat test.txt
```

## 2. 常用模块

```ansible```的模块是由```ansible```对特定部署操作脚本打包封装后的成品，可以利用该模块成品直接利用编写```PlayBooks```，这样就大大简化了日常工作中对部署脚本的编写逻辑，从而方便后期去维护管理。

### 1. File模块

用来在目标主机上创建文件或目录，并对其赋予响应的系统权限。```file```调用的是```file```模块，这里定义在目标主机上创建一个```root/foo.txt```文件。```state```表示要创建文件，```mode```表示文件```0755```权限。

```yml
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
```

### 1. Copy模块

实现```ansible```服务端到目标主机的文件传送。

```yml
- name: copy a file
  copy: 'remote_src=no src=reles/testbox/files/foo.sh' dest=/root/foo.sh mode=0644 force=yes
```

### 3. State模块

获取远程文件状态信息，并将这个信息保存在环境变量下，用于使用。保存到```script_stat```变量中。

```yml
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh'
  register: script_stat
```

### 4. Debug模块

用来在```ansibles```输出下打印数据

```yml
- debug: msg=foo.sh exists
  when: script_stat.stat.exists
```

### 5. Command/Shell模块

用来执行```Linux```目标主机命令行，```Shell```会调用```/bin/bash```, 可以使用系统变量。```Command```不可以, 推荐使用```shell```。

```yml
- name: run the script
  command: "sh /root/foo.sh"

- name: run the script
  shell: "echo 'test' > /root/test.txt"
```

### 6. Template

实现```ansible```服务端到目标主机的```jinja2```模板传统，会利用这个功能编写```app```配置文件，例如```nginx```配置文件。

```yml
- name: write the nginx config file
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx/conf
```

### 7. Packaging模块

调用目标主机系统包管理工具(```yum```, ```apt```)，根据定义的安装包的名称进行配置安装。

```yml
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest

- name: ensure nginx is at the latest version
  apt: pkg=nginx state=latest
```

### 8. Service模块

管理目标主机的系统服务

```yml
- name: start nginx service
  service: name=nginx state=started
```

## 3. 案例演示

利用上面的介绍编写一个完整的```ansible playbooks```脚本文件。

```yml
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
- name: copy a file
  copy: 'remote_src=no src=reles/testbox/files/foo.sh' dest=/root/foo.sh mode=0644 force=yes
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh'
  register: script_stat
- debug: msg=foo.sh exists
  when: script_stat.stat.exists
- name: run the script
  command: "sh /root/foo.sh"
- name: write the nginx config file
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx/conf
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest
- name: start nginx service
  service: name=nginx state=started
```

命令行连接至```ansible```服务器当中，连接至```deploy```用户中。启动```py3.6```的虚拟环境。加载```ansible2.5```版本。

```s
su -deploy

source .py3-a2.5-env/bin/activate

source .py3-a2.5-env/ansible/hacking/env-setup -q

ansible-playbook --version
```

首先需要进入到目标主机进行基本的模块配置，保证随后的任务可以正常运行。

创建两个用户```foo```和```deploy```，创建一个```nginx```目录。

```s
useradd foo
useradd deploy
mkdir /etc/nginx
```

安装```nginx```，```yum```源，保证可以安装。

```s
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

在```ansible```主机中，进入到```test_playbooks```文件夹。编辑```roles/testbox/tasks/main.yml```这个文件

```s
vi roles/testbox/tasks/main.yml
```

在文件中继续添加模块任务。

```yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
```

执行这个任务, 测试刚刚编写的任务。

```s
ansible-playbook -i inventory/testenv ./deploy.yml
```

执行之后可以在目标服务器找到这个文件。接着演示```file```

```s
mkdir roles/testbox/files/
vi roles/testbox/files/foo.sh
```

添加内容

```sh
echo "this is a test script"
```

```s
vi roles/testbox/tasks/main.yml
```

```yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
- name: copy a file
  copy: 'remote_src=no src=reles/testbox/files/foo.sh' dest=/root/foo.sh mode=0644 force=yes
```

执行这个任务, 测试刚刚编写的任务。

```s
ansible-playbook -i inventory/testenv ./deploy.yml
```

接着演示```exists```

```yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
- name: copy a file
  copy: 'remote_src=no src=reles/testbox/files/foo.sh' dest=/root/foo.sh mode=0644 force=yes
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh'
  register: script_stat
- debug: msg=foo.sh exists
  when: script_stat.stat.exists
```

```s
ansible-playbook -i inventory/testenv ./deploy.yml
```

接着添加一个模块任务，在添加模块任务之前，需要添加几个参数到```testenv```这个文件中

```s
vi inventory/testenv
```


```s
[testservers]
test.example.com

[testservers:vars]
server_name=test.example.com
user=root
output=/root/test.txt
server_name=test.example.com
port=80
user=deploy
worker+processes=4
max_open_file=65505
root=/www
```

接着需要在```roles```下面的```testbox```中添加```templates```目录。

```s
mkdir roles/testbox/templates

vi roles/testbox/templates/nginx.conf.j2
```
 
```s
# For more information on configuration, see:
user    {{ user }};
worker_processes    {{ worker_processes }};

error_log  /var/log/nginx/error.log;

pid  /var/run/nginx.pid;

evnets {
  worker_connections    {{ max_open_file }};
}

http {
  include   /etc/nginx/mime.types;
  default_type    application/octet-stream;

  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log /var/log/nginx/access.log main;

  sendfile on;

  keepalive_timeout 65;

  server {
    listen  {{ port }} default_server;

    location / {
      root  {{ root }};
      index   index.html index.htm;
    }

    error_page 404  /404.html;
    location = /404.html {
      root    /usr/share/nginx/html;
    }

    error_page    500 502 503 504 /50x.html;
    location = /50x.html {
      root    /usr/share/nginx/html
    }
  }
}
```

```s
vi roles/testbox/tasks/main.yml
```

使用```template```将本地模板编译传送至远程，然后在使用```yum```安装```nginx```，使用```service```任务启动远程的```nginx```服务。

```yml
- name: Print server name and user to remote testbox
  shell: "echo 'Currently {{ user }} is logining {{ server_name }}' > {{ output }}"
- name: create a file
  file: 'path=/root/foo.txt state=touch mode=0755 owner=foo group=foo'
- name: copy a file
  copy: 'remote_src=no src=reles/testbox/files/foo.sh' dest=/root/foo.sh mode=0644 force=yes
- name: check if foo.sh exists
  stat: 'path=/root/foo.sh'
  register: script_stat
- debug: msg=foo.sh exists
  when: script_stat.stat.exists
- name: write the nginx config file
  template: src=roles/testbox/templates/nginx.conf.j2 dest=/etc/nginx/nginx/conf
- name: ensure nginx is at the latest version
  yum: pkg=nginx state=latest
- name: start nginx service
  service: name=nginx state=started
```

```s
ansible-playbook -i inventory/testenv ./deploy.yml
```
使用远程查看命令。

```s
ssh root@test.example.com ps -ef | grep nginx
```