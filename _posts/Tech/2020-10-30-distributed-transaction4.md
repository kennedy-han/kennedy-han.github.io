---
layout: post
title: "分布式事务-Seata"
description: "分布式事务 Seata"
category: Tech
tags: [分布式事务]
---


官方：

```text
http://seata.io/zh-cn/
```



# Seata 概念



## AT模式



## 前提

- 基于支持本地 ACID 事务的关系型数据库。
- Java 应用，通过 JDBC 访问数据库。



<img src="../../../public/img/posts/25-seata01.png">

<img src="../../../public/img/posts/25-seata02.png">

<img src="../../../public/img/posts/25-seata03.png">

<img src="../../../public/img/posts/25-seata04.png">

<img src="../../../public/img/posts/25-seata05.png">



---



<img src="../../../public/img/posts/26-seata-01.png">

---



## TCC模式

<img src="../../../public/img/posts/26-seata-02.png">



## 空回滚，幂等，悬挂

<img src="../../../public/img/posts/26-seata-03.png">



---



代码参见：

https://github.com/kennedy-han/seata-all