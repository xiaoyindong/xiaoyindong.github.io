## 1. JOB构建

```jenkins```是由多个```job```任务或者```project```构成的开发系统，可以将```开发```，```测试```，```部署```，```基础运维操作```创建一个任务保存在任务列表当中，方便在```jenkins```平台中日常的维护。

使用```内建模块```或者```脚本```创建任务，在任务里面通过配置相关的```参数```以及```工具模块```作为可执行的任务，共享到```jenkins```平台下。供不同权限的人重复```build```执行。

每一次执行的结果记录称为一个```build```构建。```workspace```目录中保存```git```拉取的代码和构建之后的文件。

### 1. Freestyle job

用途最为广泛的任务类型，在配置页面添加配置项和参数就可以完成一个满足不同工作人员的需求。

缺点是每个```Freestyle job仅```能实现一个开发功能。```Freestyle```的```job```配置只能通过前台手动完成，无法通过编写一段代码去实现```Freestyle```的所有功能，所以不利于```Job配置迁移```和```版本控制```。(没有审计和历史记录)。

逻辑相对简单，无需额外学习成本。

### 2. Pipline Job

早期是```jenkins```的插件，后来集成到了```jenkins```中可直接使用。可以实现```持续集成```和```持续交付```的管道。持续集成简称```CI```,开发的每一次提交都可以自动构建。持续交付```CD```,在持续集成的基础上将打包文件部署。

所有模块， 参数配置都可以体现为一个```pipeline```脚本，可以定义多个```stage```构建一个管道工作集。所有的配置代码化，方便```Job```配置迁移与版本控制。但是有一定的学习成本，需要学习```pipline```脚本基础。

## 2. 环境配置

### 1. 配置 Jenkins server 本地 gitlab DNS

将g```itlab```所在服务器的```ip```配置到本地的```host```中。

```s
vi  /etc/hosts

# 192.168.xx.xx gitlab.example.com
```

### 2. 安装git client, curl工具依赖

```s
yum install curl -y
```

### 3. 关闭系统Git http.sslVerify安全认证

```s
git config --system http.sslVerify false
```

### 4. 添加Jenkins后台client user与email

```jenkins```后台的系统管理-系统设置，找到```Git Plugin```列表中，```name```中添加```root```，```email```中添加```root@example.com```。

### 5. 添加git Credential凭据

```jenkins```后台的凭据菜单，```jenkins``` -> ```全局凭据``` -> ```添加凭据```，类型选择```username with password```，```范围全局```，```username```输入```root```，```password```输入对应的密码，```id```和```描述```可以不填，确认即可。

## 3. freestyle构建配置

首先需要创建一个```freestyle project```, 左上角的新建项目就可以。```test-freestyle-job```, 构建一个自由风格的软件任务。

编辑描述信息，随便输入点什么就可以了。```this is first test freestyle job```。

同样在```general```中添加参数配置，在配置页面勾选参数化构建过程，添加一个选项参数和文本参数。选项参数名称为```deploy_env```, 选项为```dev```(```换行```)```prod```, 描述随便写，```choose deploy environment```。文本参数的名称填写```version```，默认值填写```1.0.0```，描述随便写，```build version```。

配置源代码管理选项打开，将```git```上的代码```clone```到```jenkins```本地，进行随后的项目部署工作。输入```gitlab```的```url```地址，这里使用```https```的地址，```credentials```选择刚刚创建的账号密码。```branch```选中```master```。

最后通过添加一个```shell```模块，完成```build```配置。在构建选项中，点击增加构建步骤，选择执行```shell```。在编辑栏中添加任务脚本。声明脚本格式。

这里面的```deploy_env```和```version```就是前面添加的参数配置，这里会将选中的值传递进来。

```s
#!/bin/sh
export PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin"

# Print env variable
echo "[INFO] Print env variable"
echo "Current deployment environment is $deploy_env" >> test.properties
echo "THe build is $version" >> test.properties
echo "[INFO] Done"

# Check test properties
echo "[INFO] Check test properties"
if [ -s test.properties ]
then
    cat test.properties
    echo "[INFO] Done..."
else
    echo "test.properties is empty"
fi

echo "[INFO] Build finished"
```

点击应用就配置完成了，点击保存自动退出配置页面。

点击左侧的````Build with Parameters````菜单，这里会出现我们之前配置的可选参数和文本参数，需要我们自己选择一下。点击开始构建。可以点击左下角的圆球，查看构建结果。

## 4. Pipline Job构建配置

代码包裹在```pipeline{}```内，```stages{}```用来包含```pipline```所有的```stage```层，用来将```pipeline```管道分为若干个管道块，每一个管道块都可以独立的做一些事情。```stage```中的```steps```子层用来添加```pipeline```的业务编写。

```s
pipline {
    agent any
    environment {
        host='test.example.com'
        user='deploy'
    }
    stages {
        stage('build') {
            steps {
                sh "cat $host"
                echo $deploy
            }
        }
    }
}
```

```agent```的作用是定义```pipeline```在哪里运行，通常可以使用```any```，```none```或具体的```jenkins node```主机名等。

```s
agent {node {label 'node1'}}
```

```environment```用来定义当前的环境变量。```变量名称=变量值```的方式，与```stages```平级就可以应用到```stages```中。如果定义在```stage```中就值可以使用在```steps```中。

```script```是可选的，定义在```steps```区域下，可以编写```groovy```脚本语言，用来进行相应的脚本逻辑运算。

```s
steps {
    echo "Hello world"
    script {
        def servers = ['node1', 'node2']
        For (int i=0; i < server.size(); ++i) {
            echo "testing ${servers[i]} server";
        }
    }
}
```

```steps```中可以使用```echo```输出，可以使用```linux```系统的```shell```命令。可以使用```git```模块进行```git```相关操作。

```s
steps {
    echo "Hello world"
    sh "cat 'hello world'"
    git url: "https://root@gitlab.example.com/root/test.git"
}
```

可以在新建任务重创建一个```Pipeline project```, 名称为```test-pipeline-job```, 类型选择流水线任务。描述信息添加```this is first pipeline job```。在```Pipeline```任务栏填写脚本。

```s
#!groovy

pipeline {
    agent {node {label 'master'}}

    environment {
        PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin"
    }

    parameters {
        choice (
            choices: 'dev\nprod',
            description: 'choose deploy environment',
            name: 'deploy_env',
        )
        string (name: 'version', defaultValue: '1.0.0', description: 'build version')
    }

    stages {
        stage("Checkout test repo") {
            steps {
                sh 'git config --global http.sslVerify false'
                dir ("${env.WORKSPACE}") {
                    git branch: 'master', credentialsId: "xxxxxxxxxx", url: 'https://root@gitlab.example.com/root/test-repo.git'
                }
            }
        }
        stage("Print env variable") {
            steps {
                dir ("${env.WORKSPACE}") {
                    sh """
                    echo "[INFO] Print env variable"
                    echo "Current deployment environment is $deploy_env" >> test.properties
                    echo "The build is $version" >> test.properties
                    echo "[INFO] Done..."
                    """
                }
            }
        }
        stage("Check test properties") {
            steps {
                dir ("${env.WORKSPACE}") {
                    sh """
                    echo "[INFO] Check test properties"
                    if [ -s test.properties ]
                    then
                        cat test.properties
                        echo "[INFO] Done..."
                    else
                        echo "test.properties is empty"
                    fi
                    """
                    echo "[INFO] Build finished..."
                }
            }
        }
    }
}
```

点击顶部```test-pipeline-job```, 重新开始```Build with Parameters```，就可以看到每个任务的执行和状态。点击蓝色圆球就可以进入到日志界面中查看了。

## 4. shell集成和jenkins参数集成

```shell```是```jenkins```中最常用的模块。

可以新建项目，名称随便起，```shell-freestyle-job```选择构建一个自由风格的项目。

描述随便填写，构建中选择执行```shell```，可以书写一些```shell```脚本。

```s
#!/bin/sh
user=`whoami`

if [ $user == 'deploy' ]
then
    echo "Hello, my name is $user"
else
    echo "Sorry I am not $user"
fi

ip addr

cat /etc/system-release

free -m

df -h

py_cmd=``which python`

$py_cmd --version
```

```Jenkins```参数是```Jenkins```任务重要的组成部分，可以通过传入不同的参数，让```Jenkins```实现不同的效果。

新建一个```parameter-freestyle-job```同样选择自由风格的软件项目。参数化构建过程。可以选择选项参数，```deploy_env```, 值为```dev,uat,stage,pord```四个值。

文本参数输入```version```，默认值输入```1.0.0```。

布尔值参数```bool```，默认值勾选，表示默认为```true```

密码参数，名称为```pass```，默认值```123456```。

在构建里面选择执行```shell```。

```s
#!/bin/sh

echo "Current deploy environment is $deploy_env"
echo "The build is $version"
echo "The password is $pass"

if $bool
then
    echo "Request is approved"
else
    echo "Request is rejected"
fi
```

## 5. Git和Maven集成

### 1. Git

使用```Jenkins```内建的```git```插件，去将```gitlab```或者```github```的代码```clone```到本地。

新建任务```git-freestyle-job```选择构建自由风格的原件项目。

在码源管理里面选择```git```，添加仓库```https```地址，```Credentials```凭证选择之前创建的```gitlab```系统凭据。保存并退出。点击立即构建就开始执行了。

### 2. Maven

使用```Jenkins```内建的```Maven```插件将从代码仓库```clone```下来的```java```源代码编译成我们事先配置好的```var```包或者```jar```包到本地，准备好随后的部署工作。

填写完```git```配置之后，构建菜单选择调用顶层```Maven```目标，输入```package```, 保存并退出。

需要配置，系统管理，全局工具配置，选择新增```JDK```，取消自动安装勾选，别名选择```jdk1.8```，```JAVA_HOME```输入安装```maven```插件时候差生的```runtime```路径复制进去。

在```Maven```中选择新增```Maven```，同样取消自动安装，```name```输入```maven-3.5.4```, 复制命令行中的```maven-home```路径粘贴到```MAVEN_HOME```中。

```apply```保存配置。找到我们创建的这个项目，点击配置，在调用顶层```Maven```目标中选择```maven-3.5.4```也就是刚刚填写的那个值。点击保存，然后开始构建。

## 6. Jenkins Ansible集成

使用```ansible```内建```shell```模块调用本地的```ansible```命令行工具，从而能够实现```Jenkins```调用```ansible```从而实现远程服务器的部署管理功能。

首先登陆```jenkins server```远程主机的命令行，并且完成了```ansible py3.6```虚拟环境配置，以及配置了```Jenkins```主机下```deploy```系统用户到```test.example.com```主机的密钥认证。


```s
su - delpoy
ll -a
cd .py3-a2.5-env
ll
# 可以发现ansible源代码的值。
```
可以使用```.py3-a2.5-env```目录进行```py3.6```环境以及```ansible2.5```环境加载工作。

查看```Jenkins```的```deploy```系统用户到```test.example.com```，不需要输入密码就可以链接。

```s
ssh root@test.example.com这台主机的密钥认证。

exit
```

```Jenkins```的```ansible```集成配置页面。

在```Jenkins```的命令行

```s
cd /home/deploy
ls
# 这里要有一个ansible的清单文件，也就是testservers文件夹
```

```testservers```文件内容, 指定服务名称和远程主机地址，以及要进入的远程主机用户名称。

```s
[testserver]
test.example.com ansible_user=root
```

回到```Jenkins```的用户界面。新建任务，```ansible-freestyle-job```，选择自由风格软件项目。描述随便填写```this is my first ansible job```，构建中选择执行```shell```。

```s
#!/bin/sh

set +x
source /home/deploy/.py3-a2.5-env/bin/activate
source /home/deploy/.py3-a2.5-env/ansible/hacking/env-setup -q

cd /home/deploy
ansible --version
ansible-playbook --version

cat testservers

ansible -i testservers testserver -m command -a "ip addr"
set -x
```