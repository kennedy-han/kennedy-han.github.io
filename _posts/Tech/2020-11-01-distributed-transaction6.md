---
layout: post
title: "分布式事务 几种解决方案"
description: "分布式事务"
category: Tech
tags: [分布式事务]
---



# 几种解决方案



## 1. 可靠消息服务

可以是自己的开发，可靠中台

<img src="../../../public/img/posts/28-transaction01.png">

<img src="../../../public/img/posts/28-transaction02.png">



## 2. 最大努力通知

某宝某信支付的回调接口，隔几分钟就会回调，尽自己最大的努力通知别人的系统，并且也提供回查接口

<img src="../../../public/img/posts/28-transaction03.png">

---



## 3. 事务消息(基于RocketMQ)

---

<img src="../../../public/img/posts/28-transaction04.png">

