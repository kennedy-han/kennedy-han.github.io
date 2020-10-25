---
layout: post
title: "分布式事务-LCN"
description: "分布式事务 LCN"
category: Tech
tags: [分布式事务]
---



官方文档：

```text
https://www.codingapi.com/docs/txlcn-setting-distributed/
```



# 原理介绍|LCN事务模式

## 一、原理介绍:

  LCN模式是通过代理Connection的方式实现对本地事务的操作，然后在由TxManager统一协调控制事务。当本地事务提交回滚或者关闭连接时将会执行假操作，该代理的连接将由LCN连接池管理。

## 二、模式特点:

- 该模式对代码的嵌入性为低。
- 该模式仅限于本地存在连接对象且可通过连接对象控制事务的模块。
- 该模式下的事务提交与回滚是由本地事务方控制，对于数据一致性上有较高的保障。
- 该模式缺陷在于代理的连接需要随事务发起方一共释放连接，增加了连接占用的时间。



# TM集群

## 1. TM启动多个

通过指定profiles 不同的端口



## 2. TC配置多个地址

```yml
tx-lcn:
  client:
    manager-address: 127.0.0.1:8071,127.0.0.1:8072
```





# 使用postman工具测试

选择 POST

地址输入 http://localhost:1001/add-order

选择Body，raw，选择 JSON

输入

```json
{
    "orderName" : "test1"
}
```

点击Send



---



<img src="../../../public/img/posts/22-lcn.png">



---



代码参见：

https://github.com/kennedy-han/lcn-parent