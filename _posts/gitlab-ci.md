---
title: Gitlab持续集成
date: 2018-03-29 23:54:13
tags: 持续集成 CI 
---

### 持续集成

前端工程化中，持续集成是非常重要的一块。具体可以看我之前的文章：[关于前端持续集成的一些理解](http://blog.excute.cn/2018/03/21/autodeploy/)

![](/images/gitlab-ci-2.png)

这里就不再赘述了

### Gitlab-Ci

Gitlab就是一个基于 Web 的 Git 仓库管理工具，从9.0版本开始提供持续集成功能（CI/CD），代码版本控制加上持续集成功能非常好用，下面具体讨论一下怎么做（假设已经配置好Gitlab）。

#### 建一个仓库，我这里取名为`test-auto-deploy`

#### 开启CI/CD功能，具体在`setting -> General`里的`permissions`中

![](/images/gitlab-ci-1.jpg)

#### 配置CI/CD。在仓库根目录中新建一个名为`.gitlab-ci.yml`文件，Gitlab将会自动识别这个文件。

这个文件包含了一些命令和CI配置，Gitlab会依据里面的内容进行操作，这里以我的配置为例：

```yml
# 语法详情可参考官方文档：https://docs.gitlab.com/ee/ci/yaml/README.html
# 设置环境
image: node:8.9.1
# job执行前都要进行的操作
before_script:
  # -
# CI执行流程：顺序执行job，一个job名可对应多个job，同一个名称的job将会并行执行
stages:
  - build
#   - test
  - deploy
# 全局缓存，stage之间共享
cache:
  paths:
    - dist/
    - node_modules/
# 打包命令,只针对master分支有效，only选项支持正则表达式
master_build:
  stage: build
  script:
    - yarn install
    - yarn run build
  only:
    - master
# 测试命令   
master_buld:
  stage: test
  script:
    - yarn test
  only:
    - master
# 部署命令
master_deploy:
  stage: deploy
  script:
    - yarn run deploy
  only:
    - master

```

这个文件也是受版本控制的，也就意味着不同的功能分支能有不同的配置，进行不同的操作。

#### 配置Runner，CI的操作是需要容器来执行的，也就是Runner。Runner分为项目私有的和可公用的，即：`Specific Runners` 和 `Shared Runners`。

Runner的安装很简单，参考官方的教程即可: [Install GitLab Runner](https://docs.gitlab.com/runner/install/)

我这里选择使用同事配置好的公用的Runner:

![](/images/gitlab-ci-3.jpg)

#### 往`master`分支推送试试看！

![](/images/gitlab-ci-4.jpg)

可以看到打包和部署两个Job都执行成功了

还可以手动执行Job和查看Job执行详情

![](/images/gitlab-ci-5.jpg)


### 总结

使用Gitlab提供的工具确实可以帮助我们提高效率，不必再自己撸一个环境。

进行操作前最好进行理论知识的学习，例如：CDN部署、持续集成、GIT、服务器基本运用、Wepack、等等。

这样就可以从重复的发布、打包中解脱出来啦，还能进行自动话测试、版本控制...

棒！
