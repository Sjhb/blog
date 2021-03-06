---
title: 关于前端持续集成的一些理解
date: 2018-03-21 22:40:05
tags: autodeploy
---

### 关于前端持续集成的一些理解
#### 什么是持续集成？
持续集成是一种标准的开发流程，简称CI，其内容为频繁的向主干分支提交代码，并持续不断地从唯一代码库同步代码。持续集成的目的就是使产品快速迭代，同时保持产品质量。

持续集成的好处有：
- 快速发现问题：频繁地提交代码，并进行测试和评审，这个过程中可以尽早发现问题今早解决，而不必积压到后期才发现问题。
- 保持分支之间的差别在一个可控的范围内：持续地与主分支代码同步，可以避免开发分支与主分差别过大导致后期集成难度大的风险。

与持续集成相关的两个重要的概念：持续交付、持续部署

这两个概念很好了解，持续交付即研发团队尽快、持续地向客服交付产品，以便更早地发现问题；持续部署即不断地将集成的代码构建、发布、自动化测试然后部署到测试环境和正式环境

#### 对应的开发流程

同步代码 =》 开发 =》 提交 =》 code review =》 自动化测试（单元、集成、端对端） =》 构架 =》 测试环境部署 =》 带上版本号,正式环境部署

如果需要的话，还要提供回滚操作

对比git flow 工作流就很好理解了

#### 落实到前端

后端部署代码还需要提到不停机更新等概念，而前端则需要注意的是非覆盖式更新、协议缓存的一些问题。

前端做自动化部署我想需要进行一下流程步骤（基于git）：

- 合并分支请求
- code review
- 合并分支并拉出release分支
- 部署webhook钩子进行自动化测试
- 将项目文件打包并发布到测试环境
- 打上版本号并发布到正式环境，保存发布文件
- 需要回滚操作时，重新发布上一次的文件

#### 集成工具

持续集成的工具有很多，例如Jenkins，Gitlab，合理利用这些工具将有效得提高我们的工作效率





