---
title: 使用 Verdaccio 搭建私有库
date: 2020-01-18 14:01:32
tags: [NPM]
categories: [Tool, NPM]
description: ""
---

## NPM 介绍

> npm 为你和你的团队打开了连接整个 JavaScript 天才世界的一扇大门。它是世界上最大的软件注册表，每星期大约有 30 亿次的下载量，包含超过 600000 个 包（package） （即，代码模块）。来自各大洲的开源软件开发者使用 npm 互相分享和借鉴。包的结构使您能够轻松跟踪依赖项和版本。

## 私有 NPM

### 需求背景

平时在项目工作中可能会用到很多通用性的代码，比如，框架类、工具类以及公用的业务逻辑代码等等，通过打包发布到 npm 中央仓库或者私有仓库，来进行维护和托管代码，方便公用代码的使用，如果涉及到不方便公开的代码组件可以发布到私有仓库。

### 私有 npm 组件库的优势

- 私有 npm 包只对公司内部局域网开放
- 速度比直接在 npm 下载包更快，也比使用淘宝镜像快，毕竟是在公司内部局域网
- 对于发布和下载 npm 包可以配置权限管理
<!-- more -->

### 同类产品对比

> 在研究搭建 npm 私有组件仓库时，查阅了很多相关方面的资料，有使用 maven 包管理的私服工具 nexus 搭建私有仓库的，也有使用阿里的 cnpm 搭建私有仓库的，还有使用 sinopia 和 verdaccio 搭建私有仓库的，下面介绍一下它们的主要区别：
>
> - [使用 maven 包管理的私服工具 nexus 搭建私有仓库](https://blog.csdn.net/chaos_hf/article/details/78668539)，这种方式功能强大，但更偏向后台使用，服务端的同学应该比较熟悉
> - [阿里的 cnpm 搭建 npm 私有仓库](https://github.com/cnpm/cnpmjs.org/wiki)，这个是阿里自己内部也在使用的，应该也挺好用，不过需要安装 MySQL 数据库，而且配置比较麻烦
> - [使用 sinopia 搭建 npm 私有仓库](https://www.jianshu.com/p/e4db4a0af96a)，这个使用比较简单，而且不用配置数据库，但是这个 github 项目好像三四年都没有人维护了<br/><br/>
>   经过查阅相关资料后对比，最后选择使用 verdaccio 来搭建 npm 私有仓库，这个 github 项目是 sinopia 的分支，用法和 sinopia 差不多。<br/><br/>
>   原文: <https://liaolongdong.com/2019/01/24/build-private-package.html>

## Verdaccio

### 简介

[Verdaccio](https://github.com/verdaccio/verdaccio) - 轻量级私有 npm 代理注册表

#### 安装与部署

#### 系统与环境

- 系统 CentOS 7.7
- 环境 node v12.13.0
- 部署 PM2 (一个带有负载均衡功能的 Node 应用的进程管理器。)

##### 安装

`npm install --global verdaccio --unsafe-perm`，因为我们使用的是 root 用户，所以需要加上`--unsafe-perm`参数

##### 启动

终端运行 `verdaccio`

```
[root@svrcnyapi ~]# verdaccio
*** WARNING: Verdaccio doesn't need superuser privileges. Don't run it under root! ***
 warn --- config file  - /root/.config/verdaccio/config.yaml
 warn --- Verdaccio started
 warn --- Plugin successfully loaded: verdaccio-htpasswd
 warn --- Plugin successfully loaded: verdaccio-audit
 warn --- http address - http://localhost:4873/ - verdaccio/4.3.4

```

此时，verdaccio 服务已经启动，但是此时只能通过 `http://localhost:4873` 访问，为此，我们需要对配置进行修改:

`vi /root/.config/verdaccio/config.yaml`

在配置末尾添加:

```
listen:
  0.0.0.0:4873
```

终端重新运行 `verdaccio`

```
[root@svrcnyapi ~]# verdaccio
*** WARNING: Verdaccio doesn't need superuser privileges. Don't run it under root! ***
 warn --- config file  - /root/.config/verdaccio/config.yaml
 warn --- Verdaccio started
 warn --- Plugin successfully loaded: verdaccio-htpasswd
 warn --- Plugin successfully loaded: verdaccio-audit
 warn --- http address - http://0.0.0.0:4873/ - verdaccio/4.3.4

```

此时，我们可以通过对应部署的服务器 ip，访问其网站，例如:`http://ip:4873`

![](https://github.com/gaoac/images-library/blob/master/blog/202001191132.png?raw=true)
上述启动方式过于简单，并且不能做到一直启动、开机自启动等功能，所以这是我们采用 pm2 来管理：

##### 部署

安装

`npm install -g pm2`

执行

`pm2 start verdaccio`

```
┌────┬─────────────────────────┬─────────┬─────────┬──────────┬────────┬──────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name                    │ version │ mode    │ pid      │ uptime │ ↺    │ status   │ cpu      │ mem      │ user     │ watching │
├────┼─────────────────────────┼─────────┼─────────┼──────────┼────────┼──────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ 1  │ verdaccio               │ N/A     │ fork    │ 13439    │ 0s     │ 0    │ online   │ 0%       │ 8.9mb    │ root     │ disabled │
│ 0  │ yapi_v1.8.5             │ 1.8.5   │ fork    │ 2436     │ 2D     │ 0    │ online   │ 0%       │ 70.6mb   │ root     │ disabled │
└────┴─────────────────────────┴─────────┴─────────┴──────────┴────────┴──────┴──────────┴──────────┴──────────┴──────────┴──────────┘

```

保证一直运行、开机自启,依次执行以下命令

`pm2 save` 获取当前运行的 Node 应用程序的快照

`pm2 startup` 产生 init 脚本 保持进程活着

至此，我们就拥有了一个稳定的私有 NPM 注册中心。

> NPM 发布相关操作命令，详见 [npm 发布包教程](https://segmentfault.com/a/1190000017461666)

### 设置 npm 源

`npm set registry http://ip:4873`

设置当前 npm 源为私有 npm 库

### 注册

`npm adduser --registry http://ip:4873`

依次输入 npm 账号用户名、密码和邮箱

### 登录

`npm login`

```➜  demo npm login
Username: gac
Password:
Email: (this IS public) gac@xxx.com
Logged in as gac on http://ip:4873/.
```

### 发布

进入将要发布的项目根目录，输入命令:

`npm publish --registry http://ip:4873`

```
npm notice === Tarball Details ===
npm notice name:          @demo/test
npm notice version:       1.0.0
npm notice package size:  5.7 MB
npm notice unpacked size: 8.7 MB
npm notice shasum:        33d09bf57d034a09dffd982bb9b5811063cf73e9
npm notice integrity:     sha512-cJOnKk639iNmR[...]E84CkzJUtkfOw==
npm notice total files:   598
npm notice
+ @demo/test@1.0.0
```

提示成功后，在浏览器中刷新 `http://ip:4873` 页面，将会看到已发布的 npm 包

![](https://github.com/gaoac/images-library/blob/master/blog/202001191325.png?raw=true)

### 下载

之后使用私有仓库中的 npm 包，就和平时 npm 操作一样了。会优先私有 npm 库中查找，如若找不到，再去其他源查找（可配置多个外部链 uplink ，默认 npmjs 官方源）

## npm 多源管理

[nrm](https://github.com/Pana/nrm) - 一个 NPM 源管理器，帮助你在不同 NPM 源中切换。

一般为了保证下载顺利，我们需要切换不同的 npm 源，这里推荐 nrm 进行管理。

### 安装

`npm install -g nrm`

### 使用

`nrm ls` - 查看当前注册的 NPM 源列表

```
nrm ls

  npm -------- https://registry.npmjs.org/
  yarn ------- https://registry.yarnpkg.com/
  cnpm ------- http://r.cnpmjs.org/
* taobao ----- https://registry.npm.taobao.org/
  nj --------- https://registry.nodejitsu.com/
  npmMirror -- https://skimdb.npmjs.com/registry/
  edunpm ----- http://registry.enpmjs.org/
```

`nrm add demo http://ip:4873/` -将我们之前部署的私有库加入 nrm，取名“demo”

再次查看 `nrm ls`,显示多出一条

```$ nrm ls

  npm -------- https://registry.npmjs.org/
  yarn ------- https://registry.yarnpkg.com/
  cnpm ------- http://r.cnpmjs.org/
* taobao ----- https://registry.npm.taobao.org/
  nj --------- https://registry.nodejitsu.com/
  npmMirror -- https://skimdb.npmjs.com/registry/
  edunpm ----- http://registry.enpmjs.org/
  demo -------- http://ip:4873/
```

`nrm use demo` - 设置当前 npm 源

```
nrm use demo


   Registry has been set to: http://ip:4873/
```

有了 nrm，我们就可以随时切换我们需要的 npm 源了。
