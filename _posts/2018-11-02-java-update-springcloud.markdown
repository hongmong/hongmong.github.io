---
layout: post
title:  "SpringCloud架构升级"
date:   2018-11-02 08:08:08
categories: framework
tags: java springcloud 微服务
excerpt: 农田管家后端服务架构升级
mathjax: false
---

* content
{:toc}

### 农田管家1.0
![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20105200.png?raw=true)

存在的问题：

* 项目中相同逻辑的代码可能存在多份，代码高度耦合
* 当一处产品逻辑变更时，需要修改多个地方，项目难以维护
* 随着产品的迭代，出现了比如：广播站，微信支付，学院。。。等产品需求，为了防止出现上述问题，我们对架构做出了升级

### 农田管家2.0
![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20105300.png?raw=true)

这次升级中我们对服务进行了拆分，实现了从代码到DB的完全隔离，使得服务易于维护，水平扩展也变得相对容易

但依然存在以下问题：

* 拆分出的服务中，当服务中的一个实例不可用时，但是服务的调用方并不知道服务已经挂了，最终会导致服务调用失败
* 当服务越来越多，如果做到对服务的监控？
* 对于每个服务中可能存在一些相同的逻辑，比如说权限校验，跨域，是否可以做统一处理？
* 当我们的服务越来越多时，服务之间的依赖会越来越多。如何监控每个服务所耗费的时间？
* 如何看到服务之间的依赖关系？

### 农田管家3.0（微服务）
最终架构如下图：

![avatar](https://github.com/hongmong/hongmong.github.io/blob/master/_posts/image/2019-03-25%20105301.png?raw=true)

