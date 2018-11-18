---
title: 使用sinopia搭建私有仓库
date: 2018-10-21 23:37:18
tags: sinopia
---

### Sinopia简单介绍

`Sinopia`是由[rlidwka](https://github.com/rlidwka)开发的具有缓存功能npm包管理工具，使用起来非常简单。通过简单的配置，即可为我们搭建私有库。

### 具体操作

> 以下操作均是在Ubantu16系统中进行

首先安装`sinopia`:

```bash
sudo yarn global add sinopia
# 或者
sudo npm i -g sinopia
```

直接运行

```bash
sinopia
```
输出：

```bash
 warn  --- config file  - /Users/wanghuan/.config/sinopia/config.yaml
 warn  --- http address - http://localhost:4873/
```

`sinopia`默认是启动本地服务并监听4873端口

这时候打开`http://localhost:4873/`可以看如下页面即说明启动成功：
![](/images/sinopia1.png)

### Sinopia配置说明

`Sinopia`的配置文件位置在每次`Sinopia`启动时都会在控制台输出，我们打开`config.yaml`可以看到默认的配置，这里对一些常用的配置进行说明。

- `storage`: npm包存放的路径

- `auth`: 用户账号相关，其有一个属性`htpasswd`,描述用户数据存储位置和是否允许注册

- `uplinks`: npm包的源，默认是npm的官网，我们可以配置为淘宝镜像，并指定超时时间

- `packages`: 包权限配置

- `logs`: 日志

- `listen`: `Sinopia`服务的地址，默认是`localhost:4873`，我们如果需要外网访问，则需要更改为`0.0.0.0:4873`，并且启动的时候使用`sudo sinopia`启动

### 发布npm包

我们发布包的时候首先要切换到我们的源，推荐使用`nrm`这个包来管理我们的源：

```bash
# 安装
yarn global add nrm
# 或者
npm i -g nrm

# 查看所有源
nrm ls
#* npm ---- https://registry.npmjs.org/
#  cnpm --- http://r.cnpmjs.org/
#  taobao - https://registry.npm.taobao.org/
#  nj ----- https://registry.nodejitsu.com/
#  rednpm - http://registry.mirror.cqupt.edu.cn/
#  npmMirror  https://skimdb.npmjs.com/registry/
#  edunpm - http://registry.enpmjs.org/
```

设置我们的私有源：

```bash
nrm add private http://47.96.10.238:999
```

登录用户，这里一定要登录，否则发布包会失败

```bash
npm adduser
# 这里需要填写用户名和密码
```

上述步骤都完成后即可开始发布我们的包了：

```bash
npm publish
```

这时候打开storage文件夹可以看到我们发布的包,其中包含tgz压缩文件和package.json描述文件:

![](/images/sinopia2.png)

打开网页即可看到包说明：

![](/images/sinopia3.png)

### 其它

一般配置私有仓库时，我们需要加一层Nginx代理。