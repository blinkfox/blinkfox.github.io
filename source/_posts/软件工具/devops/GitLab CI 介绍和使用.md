---
title: GitLab CI/CD 介绍和使用
date: 2018-11-22 23:30:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/11/22/gitlab-ci-machine.png
categories: 软件工具
tags:
  - GitLab CI
  - DevOps
  - Jenkins
---

## 一、持续集成介绍

> 持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试)来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。—— Martin Fowler

### 1 概念

- **持续集成**(`Continuous Integration`)：**频繁地(一天多次)将代码集成到主干。**让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。“持续集成并不能消除 Bug，而是让它们非常容易发现和改正。”
- **持续交付**(`Continuous Delivery`)：**频繁地将软件的新版本，交付给质量团队或者用户，以供评审。**如果评审通过，代码就进入生产阶段。持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。
- **持续部署**(`continuous Deployment`)：**代码通过评审以后，自动部署到生产环境。**是持续部署是持续交付的下一步，持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。

### 2 持续集成的好处

- **自动化构建且状态对每个人可见**。可以使用`Maven`、`Gradle`等来实现自动化构建，可以在构建过程中实现自动化测试（前提是有写单元测试用例）。集成服务器在持续集成过程中发现问题可以及时发送警告给相关的干系人。
- **解放了重复性劳动。**自动化部署工作可以解放集成、测试、部署等重复性劳动，而机器集成的频率明显比手工高很多。
- **更快地发现和修复问题。**持续集成更早的获取变更，更早的进入测试，更早的发现问题，解决问题的成本显著下降。
- **更快的交付成果。**更早发现错误减少解决错误所需的工作量。集成服务器在构建环节发现错误可以及时通知开发人员修复。集成服务器在部署环节发现错误可以回退到上一版本，服务器始终有一个可用的版本。
- **减少手工的错误。**在重复性动作上，人容易犯错，而机器犯错的几率几乎为零。
- **减少了等待时间。**缩短了从开发、集成、测试、部署各个环节的时间，从而也就缩短了中间可以出现的等待时机。持续集成，意味着开发、集成、测试、部署也得以持续。
- **更高的产品质量。**集成服务器往往提供代码质量检测等功能，对不规范或有错误的地方会进行标致，也可以设置邮件和短信等进行警告。

### 3 常用持续集成工具

- [Jenkins](https://jenkins.io/)
- [GitLab CI](https://docs.gitlab.com/ee/ci/README.html)
- [TeamCity](https://www.jetbrains.com/teamcity/)
- [Travis CI](https://www.travis-ci.org/)
- [Bamboo](https://www.atlassian.com/software/bamboo)
- [CircleCI](https://circleci.com/)
- ...

## 二、Gitlab 持续集成

![GitLab CI/CD](https://statics.sh1a.qingstor.com/2018/11/22/pipelines.png)

### 1 概念介绍

#### (1) GitLab

[GitLab](https://about.gitlab.com/) 是一个利用`Ruby on Rails`开发的开源应用程序，实现一个自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。它拥有与[GitHub](https://github.com/)类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。

#### (2) GitLab CI/CD

[GitLab CI/CD](https://docs.gitlab.com/ee/ci/README.html) 是`GitLab Continuous Integration`（Gitlab持续集成）的简称。GitLab 自`GitLab 8.0`开始提供了持续集成的功能，且对所有项目默认开启。只要在项目仓库的根目录添加`.gitlab-ci.yml`文件，并且配置了Runner（运行器），那么每一次`push`或者合并请求（`Merge Request`）都会触发[CI Pipeline](https://docs.gitlab.com/ce/ci/pipelines.html)。

#### (3) GitLab Runner

[GitLab Runner](https://docs.gitlab.com/runner/) `GitLab Runner`是一个开源项目，可以运行在 GNU / Linux，macOS 和 Windows 操作系统上。每次`push`的时候 GitLab CI 会根据`.gitlab-ci.yml`配置文件运行你流水线（`Pipeline`）中各个阶段的任务（`Job`），并将结果发送回 GitLab。GitLab Runner 是基于 Gitlab CI 的 API 进行构建的相互隔离的机器（或虚拟机）。GitLab Runner 不需要和 Gitlab 安装在同一台机器上，且考虑到 GitLab Runner 的资源消耗问题和安全问题，也不建议这两者安装在同一台机器上。

Gitlab Runner 分为三种：

- 共享Runner(`Shared runners`)
- 专享Runner(`Specific runners`)
- 分组Runner(`Group Runners`)

#### (4) Pipelines

[Pipelines](https://docs.gitlab.com/ce/ci/pipelines.html) 中文称为流水线，是分阶段执行的构建任务。如：安装依赖、运行测试、打包、部署开发服务器、部署生产服务器等流程。每一次`push`或者`Merge Request`都会触发生成一条新的Pipeline。

下面是流水线示例图：

![Pipeline Status](https://docs.gitlab.com/ce/ci/img/pipelines_index.png)

#### (5) Stages

[Stages](https://docs.gitlab.com/ce/ci/yaml/README.html#stages) 表示构建阶段，可以理解为上面所说“安装依赖”、“运行测试”等环节的流程。我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：

- 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始（当然可以在`.gitlab-ci.yml`文件中配置上一阶段失败时下一阶段也执行）
- 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
- 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

下面是一个流水线内的阶段任务示例图：

![Job Status](https://docs.gitlab.com/ce/ci/img/pipelines.png)

#### (6) Jobs

[Jobs](https://docs.gitlab.com/ce/ci/pipelines.html#jobs) 表示构建的作业（或称之为任务），表示某个 Stage 里面执行的具体任务。我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：

- 相同 Stage 中的 Jobs 无执行顺序要求，会并行执行
- 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
- 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 也失败（可以在`.gitlab-ci.yml`文件中配置允许某 Job 可以失败，也算该 Stage 成功）

#### (7) .gitlab-ci.yml

GitLab 中默认开启了 Gitlab CI/CD 的支持，且使用[YAML](http://yaml.org/)文件[.gitlab-ci.yml](https://docs.gitlab.com/ee/ci/yaml/README.html#examples)来管理项目构建配置。该文件需要存放于项目仓库的根目录（默认路径，可在 GitLab 中修改），它定义该项目的 CI/CD 如何配置。所以，我们只需要在`.gitlab-ci.yml`配置文件中定义流水线的各个阶段，以及各个阶段中的若干作业（任务）即可。

下面是`.gitlab-ci.yml`文件的一个简单的`Hello World`示例：

```yaml
# 定义 test 和 package 两个 Stages
stages:
  - test
  - package

# 定义 package 阶段的一个 job
package-job:
  stage: package
  script:
    - echo "Hello, package-job"
    - echo "I am in package stage"

# 定义 test 阶段的一个 job
test-job:
  stage: test
  script:
    - echo "Hello, test-job"
    - echo "I am in test stage"
```

以上配置中，用 stages 关键字来定义 Pipeline 中的各个构建阶段，然后用一些非关键字来定义 jobs。每个 job 中可以可以再用 stage 关键字来指定该 job 对应哪个 stage。job 里面的`script`关键字是每个 job 中必须要包含的，它表示每个 job 要执行的命令。

> **注**：猜猜上面例子的运行结果？

#### (8) Badges

[Badges](https://docs.gitlab.com/ce/ci/pipelines.html#badges) 即：**徽章**，当 Pipelines 执行过程中或者执行完成时会生成徽章，你可以将这些徽章加入到你的`README.md`文件中，便于从仓库主页看到最新的构建状态。

徽章的链接形如下：

```bash
http://example.gitlab.com/namespace/project/badges/branch/build.svg 
```

我们用 GitLab 项目的徽章作为例子，效果如下：

![Gitlab build badges](https://gitlab.com/gitlab-org/gitlab-ce/badges/master/build.svg) ![Gitlab coverage badges](https://gitlab.com/gitlab-org/gitlab-ce/badges/master/coverage.svg?job=coverage)

### 2 安装 GitLab Runner

[这里](https://docs.gitlab.com/runner/install/index.html)有 GitLab Runner安装相关的资源和文档可供大家参考。以下仅以咱们公司常用的`Centos`为例来做安装说明。

#### (1) 在线安装

```bash
# 添加官方的repo.
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash

# yum 安装Gtilab Runner.
sudo yum install gitlab-runner
```

#### (2) 离线安装

```bash
# 安装Git
sudo yum –y install git

# rpm离线安装事先下载好的 Gitlab Runner rpm包.
rpm -ivh gitlab-runner-10.5.0-1.x86_64.rpm
```

> **注**：Gitlab Runner 依赖了`Git`，所以，离线安装 Gitlab Runner 之前得首先安装Git，离线安装包可以从[这里](https://packages.gitlab.com/runner/gitlab-runner)下载。

### 3 注册 Gitlab Runner

安装了 GitLab Runner 之后,就可以为 GitLab 中的仓库[注册一个 Runner](https://docs.gitlab.com/runner/register/index.html)，注册的交互式命令如下：

```bash
sudo gitlab-runner register
```

命令的交互式的过程如下：

```bash
# 输入注册命令
sudo gitlab-runner register

# 输入公司的 GitLab 网站地址
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
http://gitlab.xxxx.com/

# 你项目仓库的token，token可以在 Settings -> CI/CD -> Runners settings 中找到.
Please enter the gitlab-ci token for this runner
xxx

# 输入描述这个 runner 的名称
Please enter the gitlab-ci description for this runner
[hostame] my-runner

# 输入 runner 的标签
Please enter the gitlab-ci tags for this runner (comma separated):
my-tag,another-tag

# 输入 runner 的执行器.
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
shell
```

以上流程注册成功之后，就可以在你的项目仓库中 `Settings` -> `CI/CD` -> `Runners settings` 看到这个 Runner 了。

### 4 Gitlab Runner 常用命令汇总

下面的表格中列出了一些常用的[Gitlab Runner命令](https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-list)，以供参考：

| 命令                     | 描述                                                         |
| ------------------------ | :----------------------------------------------------------- |
| gitlab-runner run        | 运行一个runner服务                                           |
| gitlab-runner register   | 注册一个新的runner                                           |
| gitlab-runner start      | 启动服务                                                     |
| gitlab-runner stop       | 关闭服务                                                     |
| gitlab-runner restart    | 重启服务                                                     |
| gitlab-runner status     | 查看各个runner的状态                                         |
| gitlab-runner unregister | 注销掉某个runner                                             |
| gitlab-runner list       | 显示所有运行着的runner                                       |
| gitlab-runner verify     | 检查已注册的运行程序是否可以连接到GitLab，但它不验证GitLab Runner服务是否正在使用运行程序。 |

## 三、一个Web项目 CI/CD 简单示例

接下来，用一个实际项目来演示 GitLab CI/CD 的配置和使用，其中主要包括：编译测试、项目打包、部署服务、Sonar手动检查、Sonar定时检查五个阶段。

下面用一个传统的 Java web 项目(这里称之为`cidemo`)和`Tomcat`来作为示例，并用来展示常用配置的使用。当我每次`push`代码或者`Merge Request`时，都会生成一条流水线，且会自动执行我们上面所说的一些阶段，而Sonar手动检查我们设置为手动操作，且再额外配置Sonar定时检查的任务。

> **注**：我 Gitlab Runner 是安装在`Centos`环境中，并使用的`shell`执行器。

```yaml
# 定义stages
stages:
  - test
  - install
  - run
  - sonar

# 定义安装包的存放位置和Tomcat服务器的地址的变量，便于后续部署使用.
variables:
  CIDEMO_PACKAGE_DIR: '/home/gitlab-runner/packages/cidemo/'
  SERVER_HOME_DIR: '/home/gitlab-runner/tomcat/cidemo-tomcat/'

###################### 构建编译和单元测试的job. #######################

编译测试任务:
  stage: test
  only:
    - branches
  script:
    - mvn clean test

###################### Maven安装得到war包的job. #######################

打包任务:
  stage: install
  only:
    - develop
  script:
    - mvn install
    - echo '准备将最新的war包复制、保存到某个目录里面供后续使用.'
    - rm -rf $CIDEMO_PACKAGE_DIR/*.war
    - cp target/*.war $CIDEMO_PACKAGE_DIR/cidemo.war

####################### 部署运行war包的job. #######################

部署运行任务:
  stage: run
  only:
    - develop
  script:
    - echo '准备部署和运行war包！(为了方便部署到了Tomcat中运行)'
    - cd $SERVER_HOME_DIR
    - sh bin/shutdown.sh
    - rm -rf webapps/cidemo.war
    - cp $CIDEMO_PACKAGE_DIR/cidemo.war $SERVER_HOME_DIR/webapps/cidemo.war
    - nohup sh ./bin/startup.sh > logs/cidemo_nohup.log 2>&1 &

###################### Sonar手动构建的job. #######################

Sonar手动检查:
  stage: sonar
  when: manual
  only:
    - develop
  script:
    - echo '准备对项目代码做sonar的质量检查！'
    - mvn compile && mvn sonar:sonar -Dsonar.host.url=http://172.16.34.102:9000 -Dsonar.login=497a0e0e2fc07f64c4b54edc17bb47dfa251ba34

###################### Sonar每晚定时构建的job. #######################

Sonar定时检查:
  stage: sonar
  only:
    - schedules
  script:
    - echo '开始定时对项目代码做sonar的质量检查！'
    - mvn compile && mvn sonar:sonar -Dsonar.host.url=http://172.16.34.102:9000 -Dsonar.login=497a0e0e2fc07f64c4b54edc17bb47dfa251ba34
```

## 四、Gitlab CI/CD yaml 常用配置介绍

开始构建之前`.gitlab-ci.yml`文件定义了一系列带有约束说明的任务。这些任务都是以任务名开始并且至少要包含script部分，`.gitlab-ci.yml`允许指定无限量 jobs。每个 jobs 必须有一个唯一的名字，且名字不能是下面列出的保留字段：

- `image`
- `services`
- `stages`
- `types`
- `before_script`
- `after_script`
- `variables`
- `cache`

job由一列参数来定义 jobs 的行为：

| Keyword       | Required | Description                                                  |
| ------------- | -------- | ------------------------------------------------------------ |
| script        | yes      | Runner执行的命令或脚本                                       |
| extends       | no       | 定义此作业将继承的配置条目                                   |
| image         | no       | 所使用的docker镜像，查阅[使用docker镜像](https://docs.gitlab.com/ce/ci/docker/using_docker_images.html#define-image-and-services-from-gitlab-ciyml) |
| services      | no       | 所使用的docker服务，查阅[使用docker镜像](https://docs.gitlab.com/ce/ci/docker/using_docker_images.html#define-image-and-services-from-gitlab-ciyml) |
| stage         | no       | 定义job stage（默认：`test`）                                |
| type          | no       | `stage`的别名（已弃用）                                      |
| variables     | no       | 定义job级别的变量                                            |
| only          | no       | 定义一列git分支，并为其创建job                               |
| except        | no       | 定义一列git分支，不创建job                                   |
| tags          | no       | 定义一列tags，用来指定选择哪个Runner（同时Runner也要设置tags） |
| allow_failure | no       | 允许job失败。失败的job不影响commit状态                       |
| when          | no       | 定义何时开始job。可以是`on_success`，`on_failure`，`always`或者`manual` |
| dependencies  | no       | 定义job依赖关系，这样他们就可以互相传递artifacts             |
| cache         | no       | 定义应在后续运行之间缓存的文件列表                           |
| before_script | no       | 重写一组在作业前执行的命令                                   |
| after_script  | no       | 重写一组在作业后执行的命令                                   |
| environment   | no       | 定义此作业完成部署的环境名称                                 |
| coverage      | no       | 定义给定作业的代码覆盖率设置                                 |
| etry          | no       | 定义在发生故障时可以自动重试作业的时间和次数                 |
| parallel      | no       | 定义应并行运行的作业实例数                                   |

### extends

> 是在 GitLab 11.3 中引入的。

`extends`定义了一个使用`extends`的作业将继承的条目名称。它是使用[YAML锚点](https://docs.gitlab.com/ee/ci/yaml/README.html#anchors)的替代方案，并且更加灵活和可读：

```yaml
.tests:
  script: rake test
  stage: test
  only:
    refs:
      - branches

rspec:
  extends: .tests
  script: rake rspec
  only:
    variables:
      - $RSPEC
```

在上面的示例中，`rspec`作业继承自`.tests`模板作业。 GitLab 将根据键执行反向深度合并。 GitLab将：

- 将`rspec`内容以递归方式合并到`.tests`中。
- 不合并键的值。

这实际生成的是以下`rspec`作业：

```yaml
rspec:
  script: rake rspec
  stage: test
  only:
    refs:
      - branches
    variables:
      - $RSPEC
```

> **注**: `rake test`已被`rake rspec`脚本覆盖。

### image 和 services

这两个关键字允许使用一个自定义的 Docker 镜像和一系列的服务，并且可以用于整个 job 周期。详细配置文档请查看[a separate document](https://docs.gitlab.com/ee/ci/docker/README.html)。

### before_script 和 after_script

`before_script`用来定义所有 job 之前运行的命令，`after_script`用来定义所有 job 之后运行的命令。它们可以是一个数组或者是多行字符串。

### stages

stages 用来定义可以被 job 调用的 stages。stages 的规范允许有灵活的多级 pipelines。

stages中的元素顺序决定了对应job的执行顺序：

1. 相同 stage 的 job 可以平行执行。
2. 下一个 stage 的 job 会在前一个 stage 的 job 成功后开始执行。

接下仔细看看这个例子，它包含了3个 stage：

```yaml
stages:
 - build
 - test
 - deploy
```

1. 首先，所有 build 的 jobs 都是并行执行的。
2. 所有 build 的 jobs 执行成功后，test 的 jobs 才会开始并行执行。
3. 所有 test 的 jobs 执行成功，deploy 的 jobs 才会开始并行执行。
4. 所有的 deploy 的 jobs 执行成功，`commit`才会标记为`success`。
5. 任何一个前置的 jobs 失败了，`commit`会标记为`failed`并且下一个 stages 的 jobs 都不会执行。

这有两个特殊的例子值得一提：

1. 如果`.gitlab-ci.yml`中没有定义stages，那么 job's stages 会默认定义为`build`，`test`和`deploy`。
2. 如果一个 job 没有指定 stage，那么这个任务会分配到 test stage。

### only 和 except

`only`和`except`是两个参数用分支策略来限制 jobs 构建：

- `only`定义哪些分支和标签的git项目将会被job执行。
- `except`定义哪些分支和标签的git项目将不会被job执行。

下面是refs策略的使用规则：

- only 和 except 可同时使用。如果`only`和`except`在一个 job 配置中同时存在，则以 only 为准，跳过 except(从下面示例中得出)。
- only 和 except 可以使用正则表达式。
- only 和 except 允许使用特殊的关键字：`branches`，`tags`和`triggers`。
- only 和 except 允许使用指定仓库地址但不是forks的仓库(查看示例3)。
  
在下面这个例子中，job 将只会运行以`issue-`开始的refs(分支)，然而`except`中设置将被跳过。

```yaml
job:
  # use regexp
  only:
    - /^issue-.*$/
  # use special keyword
  except:
    - branches
```

在下面这个例子中，job 将只会执行有`tags`的refs，或者通过`API`触发器明确地请求构建。

```yaml
job:
  # use special keywords
  only:
    - tags
    - triggers
```

下面这个例子将会为所有的分支执行job，但 master 分支除外。

```yaml
job:
  only:
    - branches@gitlab-org/gitlab-ce
  except:
    - master@gitlab-org/gitlab-ce
```

### variables

GItLab CI 允许在`.gitlab-ci.yml`文件中添加变量，并在 job 环境中起作用。因为这些配置是存储在 git 仓库中，所以**最好是存储项目的非敏感配置**，例如：

```yaml
variables:
  DATABASE_URL:"postgres://postgres@postgres/my_database"
```

这些变量可以被后续的命令和脚本使用。

除了用户自定义的变量外，Runner 也可以定义它自己的变量。`CI_COMMIT_REG_NAME`就是一个很好的例子，它的值表示用于构建项目的分支或tag名称。除了在`.gitlab-ci.yml`中设置变量外，还有可以通过 GitLab 的界面上设置私有变量。

这里有更多关于[variables](https://docs.gitlab.com/ce/ci/variables/README.html)的介绍。

### cache

#### cache: paths

使用`paths`指令选择要缓存的文件或目录。也可以使用通配符。

如果 cache 定义在 jobs 的作用域之外，那么它就是全局缓存，所有 jobs 都可以使用该缓存。

缓存`binaries`和`.config`中的所有文件：

```yaml
rspec:
  script: test
  cache:
    paths:
    - binaries/
    - .config
```

缓存`git`中没有被跟踪的文件：

```yaml
rspec:
  script: test
  cache:
    untracked: true
```

缓存`binaries`下没有被`git`跟踪的文件：

```yaml
rspec:
  script: test
  cache:
    untracked: true
    paths:
    - binaries/
```

job 中优先级高于全局的。下面这个`rspec` job中将只会缓存`binaries/`下的文件：

```yaml
cache:
  paths:
  - my/files

rspec:
  script: test
  cache:
    key: rspec
    paths:
    - binaries/
```

注意，缓存是在 jobs 之前进行共享的。如果你不同的 jobs 缓存不同的文件路径，必须设置不同的`cache:key`，否则缓存内容将被重写。缓存只是尽力而为之，所以别期望缓存会一直存在。

#### cache: key

`key`指令允许我们定义缓存的作用域(亲和性)，可以是所有 jobs 的单个缓存，也可以是每个 job，也可以是每个分支或者是任何你认为合适的地方。它也可以让你很好的调整缓存，允许你设置不同 jobs 的缓存，甚至是不同分支的缓存。

`cache:key`可以使用任何的[预定义变量](https://docs.gitlab.com/ce/ci/variables/README.html)。

默认key是默认设置的这个项目缓存，因此默认情况下，从GitLab 9.0开始，每个 pipelines 和 jobs 中可以共享一切。

配置示例

缓存每个job：

```yaml
cache:
  key: "$CI_JOB_NAME"
  untracked: true
```

缓存每个分支：

```yaml
cache:
  key: "$CI_COMMIT_REF_NAME"
  untracked: true
```

缓存每个 job 且每个分支：

```yaml
cache:
  key: "$CI_JOB_NAME/$CI_COMMIT_REF_NAME"
  untracked: true
```

缓存每个分支且每个stage：

```yaml
cache:
  key: "$CI_JOB_STAGE/$CI_COMMIT_REF_NAME"
  untracked: true
```

如果使用的Windows Batch(windows批处理)来跑脚本需要用%替代$：

```yaml
cache:
  key: "%CI_JOB_STAGE%/%CI_COMMIT_REF_NAME%"
  untracked: true
```

### allow_failure

`allow_failure`可以用于当你想设置一个 job 失败的之后并不影响后续的CI组件的时候。失败的 jobs 不会影响到`commit`状态。

当开启了允许 job 失败，所有的 intents 和 purposes 里的 pipeline 都是成功/绿色，但是也会有一个"`CI build passed with warnings`"信息显示在`Merge Request`或`commit`或`job page`。这被允许失败的作业使用，但是如果失败表示其他地方应采取其他（手动）步骤。

下面的这个例子中，job1和job2将会并列进行，如果job1失败了，它也不会影响进行中的下一个 stage，因为这里有设置了`allow_failure: true`。

```yaml
job1:
  stage: test
  script:
    - execute_script_that_will_fail
  allow_failure: true

job2:
  stage: test
  script:
    - execute_script_that_will_succeed

job3:
  stage: deploy
  script:
    - deploy_to_staging
```

### when

`when`用于实现在发生故障或尽管失败时运行的作业。when可以设置以下值：

- `on_success` - 只有前面 stages 的所有工作成功时才执行。这是默认值。
- `on_failure` - 当前面 stages 中任意一个jobs失败后执行。
- `always` - 无论前面 stages 中 jobs 状态如何都执行。
- `manual` - 手动执行(GitLab8.10增加)。更多请查看手动操作。

### artifacts

`artifacts`用于指定成功后应附加到 job 的文件和目录的列表。只能使用项目工作间内的文件或目录路径。在job成功完成后artifacts将会发送到GitLab中，同时也会在 GitLab UI 中提供下载。如果想要在不通的 job 之间传递`artifacts`，请查阅[依赖关系](https://docs.gitlab.com/ce/ci/yaml/README.html#dependencies)。以下是一些例子：

发送`binaries`和`.config`中的所有文件：

```yaml
artifacts:
  paths:
  - binaries/
  - .config
```

发送所有没有被Git跟踪的文件：

```yaml
artifacts:
  untracked: true
```

发送没有被Git跟踪和`binaries`中的所有文件：

```yaml
artifacts:
  untracked: true
  paths:
  - binaries/
```

## 五、其他相关内容

### 1 API触发器 Triggers

Triggers 可用于强制使用API调用重建特定分支，`tag`或`commits`。API的使用示例可以在`Settings` -> `CI/CD` -> `Pipeline triggers`中找到。

在`triggers`文档中[查看更多](https://docs.gitlab.com/ce/ci/triggers/README.html)。

### 2 配置定时任务

GitLab CI 中可以在 GitLab `Settings` -> `CI/CD` -> `Schedules`中配置定时任务，点击`New Schedule`按钮，可以配置你流水线的定时执行任务，包括：描述信息、定时的Cron表达式、目标分支、变量等信息。

然后在需要定时执行的作业的`only`分支写上`schedules`即可。

### 3 校验 .gitlab-ci.yml

GitLab CI 的每个实例都有一个名为`Lint`的嵌入式调试工具。 你可以在 GitLab 实例的`-/ci/lint`下找到该链接。

### 4 配置邮件发送

如果希望在每次构建完成后（或者在仅构建失败的情况下），想邮件发送给相关开发人员，则可以在 GitLab `Settings` -> `Integrations` 中找到`Pipelines emails`，点击进去就可以配置邮件发送相关的内容了。

### 5 GitLab Pages

[GitLab Pages](https://gitlab.com/pages/)是用于托管静态文件的服务。而`pages`是一个特殊的job，用于将静态的内容上传到GitLab，可用于为您的网站提供服务。它有特殊的语法，因此必须满足以下两个要求：

- 任何静态内容必须放在`public/`目录下
- artifacts必须定义在`public/`目录下

下面的这个例子是将所有文件从项目根目录移动到`public/`目录。`.public`工作流是`cp`，并且它不会循环复制`public/`本身。

```yaml
pages:
  stage: deploy
  script:
  - mkdir .public
  - cp -r * .public
  - mv .public public
  artifacts:
    paths:
    - public
  only:
  - master
```

更多内容请查看[GitLab Pages用户文档](https://docs.gitlab.com/ce/user/project/pages/index.html)。

### 6 跳过 jobs

如果你的`commit`信息中包含`[ci skip]`或者`[skip ci]`，不论大小写，那么这个`commit`将会创建但是 jobs 也会跳过。

---

## 参考文档

- [官方文档地址](https://docs.gitlab.com/ce/ci/yaml/README.html)
- [segmentfault yaml配置中文翻译](https://segmentfault.com/a/1190000010442764#articleHeader24)