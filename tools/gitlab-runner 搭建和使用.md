## 1. 前言

构建任务都会占用很多的系统资源，```GitLab CI```是```GitLab```的一部分，如果由```GitLab CI```来运行构建任务的话，在执行构建任务的时候，```GitLab```的性能会大幅下降。

```GitLab CI```最大的作用是管理各个项目的构建状态，因此，运行构建任务这种浪费资源的事情就交给```GitLab Runner```来做。

```GitLab Runner```最好安装到单独的linux机器上，在构建任务运行期间并不会影响到```GitLab```的性能。

在```GitLab Runner 10```中，名称从```GitLab ci multi Runner```变为```GitLab Runner```。如果想要了解```GitLab ci multi Runner```可查看```https://docs.gitlab.com/runner/install/old.html```

## 2. 安装Docker

如果要使用```Docker executor```，需要在```GitLab Runner```之前安装Docker。如果不使用此步骤可跳过。

```s
curl -sSl https://get.docker.com/ | sh
```

## 3. 安装 gitlab ci runner

官方文档：```GitLab Runner(https://docs.gitlab.com/runner/install/linux-repository.html)```

```GitLab Runner```其实是将某台服务器，注册成为```GitLab CI```的任务执行单元，用于在```CI```过程中，执行相应的任。

```Gitlab CI```对```Runner```并没有什么严格的要求，可以是一台```linux```实体机，也可以是个```Docker```容器。

针对不同的```Runner```，可以在注册时划分不同的固有角色（https://docs.gitlab.com/runner/executors/index.html），以及打上不同的自定义标签，以便于```GitLab CI```在调度时灵活分配任务，多个```Runner```并行执行任务等等。

```GitLab```针对免费用户，提供了每个月```2000```分钟的免费构建时间，在不超过这个时间时，可以直接使用```GitLab```提供的共享```Runner```。但是使用别人的服务器来进行构建任务，总是会暴露一些敏感信息出去，所以自建```GitLab Runner```，就是最佳选择了。

```s

# 添加yum源
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash

```

- 安装最新版gitlab-runner

```s
sudo yum install gitlab-runner
```

- 安装指定版本的gitlab-runner

```s
# 查看列表
yum list gitlab-runner --showduplicates | sort -r
# 安装10.0.0-1
sudo yum install gitlab-runner-10.0.0-1
```

- 查看是否正常运行

```s
gitlab-runner status
```

## 4. 注册

注册将```runner```与一个或多个```GitLab```实例绑定的过程，实例是指```整个gitlab```，```group组```，```某个仓库```。

通过重复执行```register```命令，可以在同一台主机上注册多个运行程序，每个运行程序具有不同的配置。也就是可以生成多个```runner```。

```s
sudo gitlab-runner register
```

#### 1. Please enter the gitlab-ci coordinator URL

输入要注册的```gitla```b网站的地址，必须是```http```或者```https```路径。可以是域名也可以是```ip```。

#### 2. Please enter the gitlab-ci token for this runner

输入```token```，可以是具体某个工程的```token```，也可以是某个组的```token```或者整个```gitlab```的```token```。

- gitlab全仓库token需要administrator账号登录，Admin -> Overview -> Runners找到

- group runner 可在组Settings -> CI/CD -> expand -> Runners找到

- project runner，可在项目Settings > CI/CD -> expand -> Runners找到

#### 3. Please enter the gitlab-ci description for this runner

输入runner的描述信息，随便填写或者不填写都可以，主要用于标识。

#### 4. Please enter the gitlab-ci tags for this runner

给```runner```指派```tags```，之后可以在```Gitlab```网页的```setting```上修改, 这里的```tags```就是```yml```文件中指定的```tags```。

#### 5. Whether to run untagged builds

选择```runner```是否接收未指定```tags```的任务。

#### 5. Whether to lock Runner to current project：

选择是否为当前项目锁定该 ```Runner```，默认```false```，直接回车就可以。

#### 6. Please enter the executor

选择```runner```类型，包括多种，可以是```docker```，也可以是```shell```，根据自己的需要进行选择。```(docker-ssh, shell, ssh, virtualbox, docker+machine, custom, parallels, docker-ssh+machine, kubernetes, docker)```这里我们选择的是```shell```。选择```docker```之后需要下一步配置这个```runner```默认的根镜像

#### 7. Please enter the default Docker image (e.g. ruby:2.6):

选择```docker```之后，需要配置这个```runner```默认的根镜像, 默认是ruby

## 4. 设置docker权限

为了能让```gitlab-runner```正确的执行```docker```命令，需要把```gitlab-runner```用户添加到```docker group```里，然后重启```docker```和```gitlab ci runner```。

```s
# 添加
usermod -aG docker gitlab-runner
# 重启
service docker restart
# 重启
gitlab-runner restart
```

## 5. 执行ci/cd

```pipelines```是一个构件的过程，想要使用的话需要在项目中有一个有一个```.gitlab-ci.yml```文件。

如下这个文件里面定义了两个阶段```build```和```test```，```test```和```build```里面的```scrip```t就是脚本命令。

```yml
# 定义stages
stages:
    - build
    - test
# 定义job
job1:
    stage: test
    script:
        - echo "I am job1"
        - echo "I am in test stage"
# 定义job
job2:
    stage: build
    script:
        - echo "I am job2"
        - echo "I am in build stage"
```

这个时候```pipelines```菜单可是一个```pending```状态，还需要给任务添加```docker```容器，在```job```里面添加```tags```，在```tags```里面指定容器的名字。这里的tags就是上面指定的tag。

```yml
# 定义stages
stages:
    - build
    - test
# 定义job
job1:
    stage: test
    tags:
        - demo
    script:
        - echo "I am job1"
        - echo "I am in test stage"
# 定义job
job2:
    stage: build
    tags:
        - demo
    script:
        - echo "I am job2"
        - echo "I am in build stage"
```

这里的stage是有状态的，而且会按顺序执行，前面如果失败后面就不会继续执行了。

## 6. 修改host

确保两台机器可以```ping```通，```gitlab-runner```会通过```gitlab.example.com```域名访问```gitlab-ci```，如无域名可通过修改```host```完成。

```s
vi  /etc/hosts

# 192.168.xx.xx gitlab.example.com
```

## 7. 更新gitlab runner

```s
sudo yum update
sudo yum install gitlab-runner
```

## 8. 设置官方镜像源

```s
cat <<EOF | sudo tee /etc/apt/preferences.d/pin-gitlab-runner.pref
Explanation: Prefer GitLab provided packages over the Debian native ones
Package: gitlab-runner
Pin: origin packages.gitlab.com
Pin-Priority: 1001
EOF
```
