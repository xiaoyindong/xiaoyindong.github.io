## 1. 概述

官方文档: https://docs.gitlab.com/ce/ci/variables/README.html

当```GitLab CI```接受到一个```job```后，```Runner```就开始准备构建环境。开始设置预定义的变量(环境变量)和用户自定义的变量。

变量可以被重写，并且是按照下面的顺序进行执行：

1. Trigger variables(优先级最高)

2. Secret variables

3. YAML-defined job-level variables

4. YAML-defined global variables

5. Deployment variables

6. Predefined variables (优先级最低)

例如如果你定义了私有变量```API_TOKEN=secure```，并且在```.gitlab-ci.yml```中定义了```API_TOKEN=yaml```，那么私有变量```API_TOKEN```的值将是```secure```，因为```secret variables```的优先级较高。

## 2. 预设变量

有部分预定义的环境变量仅仅只能在最小版本的```GitLab Runner```中使用。请参考下表查看对应的```Runner```版本要求。

注意：```从GitLab 9.0```开始，部分变量已经不提倡使用。请查看9.0Renaming部分来查找他们的替代变量。强烈建议使用新的变量，我们也会在将来的GitLab版本中将他们移除。

| Variable | gitlab version | Runner | Description |
| ---- | ---- | ---- | ---- |
| CI | all | 0.4 | 标识该job是在CI环境中执行 |
| CI_COMMIT_REF_NAME | 9.0 | all | 用于构建项目的分支或tag名称 |
| CI_COMMIT_REF_SLUG | 9.0 | all | 先将$CI_COMMIT_REF_NAME的值转换成小写，最大不能超过63个字节，然后把除了0-9和a-z的其他字符转换成-。在URLs和域名名称中使用。|
| CI_COMMIT_SHA | 9.0 | all | commit的版本号 |
| CI_COMMIT_TAG | 9.0 | 0.5 | commit的tag名称。只有创建了tags才会出现。 |
| CI_DEBUG_TRACE | 9.0 | 1.7 | debug tracing开启时才生效 |
| CI_ENVIRONMENT_NAME | 8.15 | all | job的环境名称 |
| CI_ENVIRONMENT_SLUG | 8.15 | all | 环境名称的简化版本，适用于DNS，URLs，Kubernetes labels等 |
| CI_JOB_ID | 9.0 | all | GItLab CI内部调用job的一个唯一ID |
| CI_JOB_MANUAL | 8.12 | all | 表示job启用的标识 |
| CI_JOB_NAME | 9.0 | 0.5 | .gitlab-ci.yml中定义的job的名称 |
| CI_JOB_STAGE | 9.0 | 0.5 | .gitlab-ci.yml中定义的stage的名称 |
| CI_JOB_TOKEN | 9.0 | 1.2 | 用于同GitLab容器仓库验证的token |
| CI_REPOSITORY_URL | 9.0 | all | git仓库地址，用于克隆 |
| CI_RUNNER_DESCRIPTION | 8.10 | 0.5 | GitLab中存储的Runner描述 |
| CI_RUNNER_ID | 8.10 | 0.5 | Runner所使用的唯一ID |
| CI_RUNNER_TAGS | 8.10 | 0.5 | Runner定义的tags |
| CI_PIPELINE_ID | 8.10 | 0.5 | GitLab CI 在内部使用的当前pipeline的唯一ID |
| CI_PIPELINE_TRIGGERED | all | all | 用于指示该job被触发的标识 |
| CI_PROJECT_DIR | all | all | 仓库克隆的完整地址和job允许的完整地址 |
| CI_PROJECT_ID | all | all | GitLab CI在内部使用的当前项目的唯一ID |
| CI_PROJECT_NAME | 8.10 | 0.5 | 当前正在构建的项目名称（事实上是项目文件夹名称） |
| CI_PROJECT_NAMESPACE | 8.10 | 0.5 | 当前正在构建的项目命名空间（用户名或者是组名称） |
| CI_PROJECT_PATH | 8.10 | 0.5 | 命名空间加项目名称 | 
| CI_PROJECT_PATH_SLUG | 9.3 | all | $CI_PROJECT_PATH小写字母、除了0-9和a-z的其他字母都替换成-。用于地址和域名名称。 |
| CI_PROJECT_URL | 8.10 | 0.5 | 项目的访问地址（http形式） |
| CI_REGISTRY | 8.10 | 0.5 | 如果启用了Container Registry，则返回GitLab的Container Registry的地址 | 
| CI_REGISTRY_IMAGE | 8.10 | 0.5 | 如果为项目启用了Container Registry，它将返回与特定项目相关联的注册表的地址 |
| CI_REGISTRY_PASSWORD | 9.0 | all | 用于push containers到GitLab的Container Registry的密码 |
| CI_REGISTRY_USER | 9.0 | all | 用于push containers到GItLab的Container Registry的用户名 |
| CI_SERVER | all | all | 标记该job是在CI环境中执行 |
| CI_SERVER_NAME | all | all | 用于协调job的CI服务器名称 |
| CI_SERVER_REVISION | all | all | 用于调度job的GitLab修订版 |
| CI_SERVER_VERSION | all | all | 用于调度job的GItLab版本 |
| ARTIFACT_DOWNLOAD_ATTEMPTS | 8.15 | 1.9 | 尝试运行下载artifacts的job的次数 |
| GET_SOURCES_ATTEMPTS | 8.15 | 1.9 | 尝试运行获取源的job次数 |
| GITLAB_CI | all | all | 用于指示该job是在GItLab CI环境中运行 |
| GITLAB_USER_ID | 8.12 | all | 开启该job的用户ID |
| GITLAB_USER_EMAIL | 8.12 | all | 开启该job的用户邮箱 |
| GITLAB_USER_NAME | 8.12 | all | 开启该job的用户名称 |
| RESTORE_CACHE_ATTEMPTS | 8.15 | 1.9 | 尝试运行存储缓存的job的次数 |


### 8.x被移除的变量

| 8.x name | 9.0+ name |
| ---- | ---- |
| CI_BUILD_ID | CI_JOB_ID |
| CI_BUILD_REF | CI_COMMIT_SHA |
| CI_BUILD_TAG | CI_COMMIT_TAG |
| CI_BUILD_REF_NAME | CI_COMMIT_REF_NAME |
| CI_BUILD_REF_SLUG | CI_COMMIT_REF_SLUG |
| CI_BUILD_NAME | CI_JOB_NAME |
| CI_BUILD_STAGE | CI_JOB_STAGE |
| CI_BUILD_REPO | CI_REPOSITORY_URL |
| CI_BUILD_TRIGGERED | CI_PIPELINE_TRIGGERED |
| CI_BUILD_MANUAL | CI_JOB_MANUAL |
| CI_BUILD_TOKEN | CI_JOB_TOKEN |

## 3. 自定义变量

```GitLab Runner0.5```或更高版本，并且```GitLab CI 7.14```更高版本支持自定义变量。

```.gitlab-ci.yml```中可以添加自定义变量，这个变量在构建环境中设置。

如果将变量设置为全局下，则它将用于所有执行的命令脚本中。

```s
variables:
  DATABASE_URL: "postgres://postgres@postgres/my_database"
```

```YAML```中定义的变量也将应用到所有创建的服务容器中，因此可以对它进行微调。

变量可以定义为全局，同时也可以定义为job级别。若要关闭全局定义变量，请定义一个空```{}```

```s
job_name:
  variables: {}
```

变量定义中可以使用其他变量

```s
variables:
  LS_CMD: 'ls $FLAGS $$TMP_DIR'
  FLAGS: '-al'
script:
  - 'eval $LS_CMD'  # will execute 'ls -al $TMP_DIR'
```

## 4. 私有变量

```GitLab Runner0.4.0```或更高版本，支持。

私有变量不会隐藏，如果明确要这么做，他们的值可以显示在```job```日志中。

如果项目是公共的或内部的，可以在项目的```pipeline```中设置```pipeline```为私有的。

私有变量存储在```.gitlab-ci.yml```中，并被安全的传递给```GitLab Runner```。建议使用该方法存储诸如```密码```、```秘钥```和```凭据```之类的东西。

可在```Settings``` -> ```Pipelines```中增加私有变量，一旦设置，所有的后续pipeline是都可以使用。

## 5. 变量保护

此功能要求GitLab 9.3或更高版本。

私有变量可以被保护。当一个私有变量被保护时，它只会安全的传递到在受保护的分支或受保护的标签上运行的```pipeline```。其他的```pipeline```将不会得到该变量。

可用在私有变量时，添加```Protected```。

## 6. 使用变量

在构建环境变量时，所有的变量都会被设置为环境变量，可以使用普通方法访问这些变量。

在大多数情况下，用于执行```job```脚本都是通过```bash```或者是```sh```。

不同环境下使用方式不同。

| Shell | 用法 |
| ---- | ---- |
| bash/sh | $variable |
| windows batch | %variable% |
| PowerShell | $env:variable |

在```bash```中访问环境变量，需要给变量名称加上前缀```$```：

```s
job_name:
  script:
    - echo $CI_JOB_ID
```

在```Windows```系统的```PowerShell```中访问环境变量，需要给变量名称加上前缀```$env:```

```s
job_name:
  script:
    - echo $env:CI_JOB_ID
```

可以使用```export```命令来列出所有的环境变量。在使用此命令时要注意会在```job```记录中列出所有私有变量的值

```s
job_name:
  script:
    - export
```

日志如下:

```s
export CI_JOB_ID="50"
export CI_COMMIT_SHA="1ecfd275763eff1d6b4844ea3168962458c9f27a"
export CI_COMMIT_REF_NAME="master"
export CI_REPOSITORY_URL="https://gitlab-ci-token:abcde-1234ABCD5678ef@example.com/gitlab-org/gitlab-ce.git"
export CI_COMMIT_TAG="1.0.0"
export CI_JOB_NAME="spec:other"
export CI_JOB_STAGE="test"
export CI_JOB_MANUAL="true"
export CI_JOB_TRIGGERED="true"
export CI_JOB_TOKEN="abcde-1234ABCD5678ef"
export CI_PIPELINE_ID="1000"
export CI_PROJECT_ID="34"
export CI_PROJECT_DIR="/builds/gitlab-org/gitlab-ce"
export CI_PROJECT_NAME="gitlab-ce"
export CI_PROJECT_NAMESPACE="gitlab-org"
export CI_PROJECT_PATH="gitlab-org/gitlab-ce"
export CI_PROJECT_URL="https://example.com/gitlab-org/gitlab-ce"
export CI_REGISTRY="registry.example.com"
export CI_REGISTRY_IMAGE="registry.example.com/gitlab-org/gitlab-ce"
export CI_RUNNER_ID="10"
export CI_RUNNER_DESCRIPTION="my runner"
export CI_RUNNER_TAGS="docker, linux"
export CI_SERVER="yes"
export CI_SERVER_NAME="GitLab"
export CI_SERVER_REVISION="70606bf"
export CI_SERVER_VERSION="8.9.0"
export GITLAB_USER_ID="42"
export GITLAB_USER_EMAIL="user@example.com"
export CI_REGISTRY_USER="gitlab-ci-token"
export CI_REGISTRY_PASSWORD="longalfanumstring"
```

## 7. Deploment variables

注意：此功能要求GitLab CI 8.15或者更高版本。

负责部署配置的项目服务可以定义在构建环境中设置自己的变量。这些变量只定义用于部署job。请参考您正在使用的项目服务的文档，以了解他们定义的变量。

一个定义有部署变量的项目服务示例Kubernetes Service。

## 8. 调试

启用调试跟踪可能会带来重的安全隐患。输出内容将包含所有的私有变量和其他的隐私，输出的内容将被上传到GitLab服务器并且将会在job记录中明显体现。

默认情况下，```GitLab Runner```会隐藏了处理```job```时正在做的大部分细节。这种行为使```job```跟踪很短，并且防止秘密泄露到跟踪中，除非您的脚本将他们输出到屏幕中。

如果```job```没有按照预期的运行，这也会让问题查找变得更加困难；在这种情况下，可以在```.gitlab-ci.yml```中开启调试记录。它需要```GitLab Runner v1.7```版本以上，此功能会启用```shell```的执行记录，从而产生详细的```job```记录，列出所有执行的命令，设置变量等。

在启用此功能之前，建议确保job只对团队成员可见。您也应该```https://docs.gitlab.com/ce/ci/pipelines.html#seeing-build-status```所有生成的```job```记录，然后使其可见。

```.gitlab-ci.yml```文件中设置```CI_DEBUG_TRACE```变量的值为```true```来开启调试记录。


```s
job_name:
  variables:
    CI_DEBUG_TRACE: "true"
```

输出示例如下：

```s
...
export CI_SERVER_TLS_CA_FILE="/builds/gitlab-examples/ci-debug-trace.tmp/CI_SERVER_TLS_CA_FILE"
if [[ -d "/builds/gitlab-examples/ci-debug-trace/.git" ]]; then
  echo $'\''\x1b[32;1mFetching changes...\x1b[0;m'\''
  $'\''cd'\'' "/builds/gitlab-examples/ci-debug-trace"
  $'\''git'\'' "config" "fetch.recurseSubmodules" "false"
  $'\''rm'\'' "-f" ".git/index.lock"
  $'\''git'\'' "clean" "-ffdx"
  $'\''git'\'' "reset" "--hard"
  $'\''git'\'' "remote" "set-url" "origin" "https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@example.com/gitlab-examples/ci-debug-trace.git"
  $'\''git'\'' "fetch" "origin" "--prune" "+refs/heads/*:refs/remotes/origin/*" "+refs/tags/*:refs/tags/*"
else
  $'\''mkdir'\'' "-p" "/builds/gitlab-examples/ci-debug-trace.tmp/git-template"
  $'\''rm'\'' "-r" "-f" "/builds/gitlab-examples/ci-debug-trace"
  $'\''git'\'' "config" "-f" "/builds/gitlab-examples/ci-debug-trace.tmp/git-template/config" "fetch.recurseSubmodules" "false"
  echo $'\''\x1b[32;1mCloning repository...\x1b[0;m'\''
  $'\''git'\'' "clone" "--no-checkout" "https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@example.com/gitlab-examples/ci-debug-trace.git" "/builds/gitlab-examples/ci-debug-trace" "--template" "/builds/gitlab-examples/ci-debug-trace.tmp/git-template"
  $'\''cd'\'' "/builds/gitlab-examples/ci-debug-trace"
fi
echo $'\''\x1b[32;1mChecking out dd648b2e as master...\x1b[0;m'\''
$'\''git'\'' "checkout" "-f" "-q" "dd648b2e48ce6518303b0bb580b2ee32fadaf045"
'
+++ hostname
++ echo 'Running on runner-8a2f473d-project-1796893-concurrent-0 via runner-8a2f473d-machine-1480971377-317a7d0f-digital-ocean-4gb...'
Running on runner-8a2f473d-project-1796893-concurrent-0 via runner-8a2f473d-machine-1480971377-317a7d0f-digital-ocean-4gb...
++ export CI=true
++ CI=true
++ export CI_DEBUG_TRACE=false
++ CI_DEBUG_TRACE=false
++ export CI_COMMIT_SHA=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ CI_COMMIT_SHA=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ export CI_COMMIT_BEFORE_SHA=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ CI_COMMIT_BEFORE_SHA=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ export CI_COMMIT_REF_NAME=master
++ CI_COMMIT_REF_NAME=master
++ export CI_JOB_ID=7046507
++ CI_JOB_ID=7046507
++ export CI_REPOSITORY_URL=https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@example.com/gitlab-examples/ci-debug-trace.git
++ CI_REPOSITORY_URL=https://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@example.com/gitlab-examples/ci-debug-trace.git
++ export CI_JOB_TOKEN=xxxxxxxxxxxxxxxxxxxx
++ CI_JOB_TOKEN=xxxxxxxxxxxxxxxxxxxx
++ export CI_PROJECT_ID=1796893
++ CI_PROJECT_ID=1796893
++ export CI_PROJECT_DIR=/builds/gitlab-examples/ci-debug-trace
++ CI_PROJECT_DIR=/builds/gitlab-examples/ci-debug-trace
++ export CI_SERVER=yes
++ CI_SERVER=yes
++ export 'CI_SERVER_NAME=GitLab CI'
++ CI_SERVER_NAME='GitLab CI'
++ export CI_SERVER_VERSION=
++ CI_SERVER_VERSION=
++ export CI_SERVER_REVISION=
++ CI_SERVER_REVISION=
++ export GITLAB_CI=true
++ GITLAB_CI=true
++ export CI=true
++ CI=true
++ export GITLAB_CI=true
++ GITLAB_CI=true
++ export CI_JOB_ID=7046507
++ CI_JOB_ID=7046507
++ export CI_JOB_TOKEN=xxxxxxxxxxxxxxxxxxxx
++ CI_JOB_TOKEN=xxxxxxxxxxxxxxxxxxxx
++ export CI_COMMIT_REF=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ CI_COMMIT_REF=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ export CI_COMMIT_BEFORE_SHA=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ CI_COMMIT_BEFORE_SHA=dd648b2e48ce6518303b0bb580b2ee32fadaf045
++ export CI_COMMIT_REF_NAME=master
++ CI_COMMIT_REF_NAME=master
++ export CI_COMMIT_NAME=debug_trace
++ CI_JOB_NAME=debug_trace
++ export CI_JOB_STAGE=test
++ CI_JOB_STAGE=test
++ export CI_SERVER_NAME=GitLab
++ CI_SERVER_NAME=GitLab
++ export CI_SERVER_VERSION=8.14.3-ee
++ CI_SERVER_VERSION=8.14.3-ee
++ export CI_SERVER_REVISION=82823
++ CI_SERVER_REVISION=82823
++ export CI_PROJECT_ID=17893
++ CI_PROJECT_ID=17893
++ export CI_PROJECT_NAME=ci-debug-trace
++ CI_PROJECT_NAME=ci-debug-trace
++ export CI_PROJECT_PATH=gitlab-examples/ci-debug-trace
++ CI_PROJECT_PATH=gitlab-examples/ci-debug-trace
++ export CI_PROJECT_NAMESPACE=gitlab-examples
++ CI_PROJECT_NAMESPACE=gitlab-examples
++ export CI_PROJECT_URL=https://example.com/gitlab-examples/ci-debug-trace
++ CI_PROJECT_URL=https://example.com/gitlab-examples/ci-debug-trace
++ export CI_PIPELINE_ID=52666
++ CI_PIPELINE_ID=52666
++ export CI_RUNNER_ID=1337
++ CI_RUNNER_ID=1337
++ export CI_RUNNER_DESCRIPTION=shared-runners-manager-1.example.com
++ CI_RUNNER_DESCRIPTION=shared-runners-manager-1.example.com
++ export 'CI_RUNNER_TAGS=shared, docker, <a href="http://www.ttlsa.com/linux/" title="linux"target="_blank">linux</a>, ruby, mysql, postgres, mongo'
++ CI_RUNNER_TAGS='shared, docker, linux, ruby, mysql, postgres, mongo'
++ export CI_REGISTRY=registry.example.com
++ CI_REGISTRY=registry.example.com
++ export CI_DEBUG_TRACE=true
++ CI_DEBUG_TRACE=true
++ export GITLAB_USER_ID=42
++ GITLAB_USER_ID=42
++ export GITLAB_USER_EMAIL=user@example.com
++ GITLAB_USER_EMAIL=user@example.com
++ export VERY_SECURE_VARIABLE=imaverysecurevariable
++ VERY_SECURE_VARIABLE=imaverysecurevariable
++ mkdir -p /builds/gitlab-examples/ci-debug-trace.tmp
++ echo -n '-----BEGIN CERTIFICATE-----
MIIFQzCCBCugAwIBAgIRAL/ElDjuf15xwja1ZnCocWAwDQYJKoZIhvcNAQELBQAw'
...
```
