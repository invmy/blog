---
title: interactivebrokers - IBKR API
description: IBKR API的开始，如何选择？少踩坑，最佳实践来说各种API的优缺点
date: 2026-07-14
tags:
  - IBKR
authors:
  - src/content/authors/enscribe.md
image: ../../../assets/hero-ibkr-home-xl.png
draft: false
---
在 [IBKRAPI](https://www.interactivebrokers.com/campus/ibkr-api-page/) 页面中可以分为三种API，

- Web API 标准的REST API端点
- TWS API 基于本地 Socket 通讯的异步事件驱动（Asynchronous Event-Driven）架构
- Flex API 报表表格专用，每日更新

### Web API

不要被官方文档迷惑 web api 1.0是旧版，而WEB API才是新版，不过目前新版WS还没更新，使用1.0API就行了，后续可以直接迁移的。这个API就是标准的REST API端点，通过不同的HTTP请求方法和端点来进行操作、还有WS进行实时行情推送。通过[Oauth 1.0a](https://www.interactivebrokers.com/campus/ibkr-api-page/cpapi-v1/#oauth-10a)可以实现免网关使用。下面展示两种链接方法



#### Oauth 1.0a

不用运行官方的网关，而是自己生成Oauth直接链接ibkr服务器。需要自己生成对应的token和证书提交公钥给ibkr，这属于第一方验证，只能自己使用。[Voyz/ibind/wiki/OAuth-1.0a](https://github.com/Voyz/ibind/wiki/OAuth-1.0a) 可以看一下，写的很详细。同时还有一个项目是部署到cloudflare workers上实现真正的个人网关。[IBKR_Gateway-workers](https://github.com/invmy/IBKR_Gateway-workers)  只要配置了对应的环境变量，不用管复杂的Oauth1.0a密钥生成了。



#### 官方Gateway

直接运行在本地运行官方的网关，官方使用了SSO 验证，通过自己的账户和密码登录后。网关会在本地运行一个反向代理，所有的请求均会转发到 `api.ibkr.com` 这个服务器使用akamai作为CDN分发。在大陆的连通性惨不忍睹，只有移动的入口延迟低一点。网关的root文件夹中有个webapps可以在这里写前端HTML+js，这样就不用再运行和进行复杂的链接维持了。



TWS API

只能通过官方的gateway登录。属于是非常难用的一档了，基于事件的框架。需要自己实现 链接管理、事件触发、事件回调。

EClient (发送者)： 这是你的程序用于向 TWS 发送指令的“客户端 socket”。比如 placeOrder, reqMktData（请求行情数据）。

EWrapper (接收者)： 这是最重要的部分，即接口回调层。你需要实现这个接口，用于定义当 TWS 推送数据回来时，你的程序该如何响应。

不过不用担心，现在有非常多的非官方项目。实现标准的同步回调可以使用await直接获取对应的信息。

- node.js版 [https://github.com/stoqey/ibkr](https://github.com/stoqey/ibkr)
- python版 [https://github.com/ib-api-reloaded/ib_async](https://github.com/ib-api-reloaded/ib_async)

选择node.js版还是python？强烈推荐node.js版本，因为已经封装了大部分需要用的api同时还有APInext可以用。几乎大部分端点直接await就能获取。同时搭配bun的elysiajs，快速构建出api服务器。



Flex API

一个只读的报表API。没什么好讲的。需要切换到英文语言界面才会有flex api获取[https://www.interactivebrokers.com/campus/ibkr-api-page/flex-web-service/](https://www.interactivebrokers.com/campus/ibkr-api-page/flex-web-service/)



## 如何选择

如果你追求极致的速度和执行速度，选择TWS API。超快的TCP链接速度，缺点是必须要购买数据包且开发难度较大，不过有了@stoqey/ibkr 那开发几乎很简单就能开发好了。



如果你想快速开发，熟练使用前端技术栈，那就用Web API。缺点是http天生缺陷，握手延迟较大。同时频率限制也严重。友好的一点是，通过ws接收到的quotes是实时数据不用买数据包，但是通过端点获取的历史数据是延时15分钟的。



开发愉快...

