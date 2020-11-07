---
layout: post
title: "分布式事务 事务消息"
description: "分布式事务"
category: Tech
tags: [分布式事务]
---




## 事务消息

### 基于RocketMQ的事务消息实现

<img src="../../../public/img/posts/29-transaction0.png">

---



rocketmq-all-4.5.0-bin-release

rocketmq-externals



启动顺序

namesrv

```sh
 start mqnamesrv.cmd
```



broker

```sh
start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true
```



externals

```sh
java -jar rocketmq-console-ng-1.0.1.jar
```



需要修改配置：

rocketmq-externals/rocketmq-console/src/main/resources/application.properties中

```
server.port=8080

rocketmq.config.namesrvAddr=localhost:9876
```



代码地址：

```
https://github.com/kennedy-han/rocket-tx
```





### 总结分布式事务

2pc(协调者超时 回滚，占用连接，效率低)

3pc (2pc的第一阶段 拆成了 2个阶段，协调者和参与者都超时，pre超时是回滚，do 超时是提交)。

tcc（2pc的第二阶段 拆成了2个阶段，不占用连接，性能高，但是麻烦）(简单业务可以tcc)。

lcn(lcn,tcc)(代码)

seata(at,tcc)（代码）

消息队列+本地事件表（代码）

最大努力通知（举例：某宝某信回调接口）

可靠消息服务（举例：中台）

消息事务（代码）



## 

---

|        | 2pc  | tcc  | 消息队列 |      |
| ------ | ---- | ---- | -------- | ---- |
| 一致性 | 强   | 最终 | 最终     |      |
| 吞吐量 | 低   | 中等 | 高       |      |
| 复杂度 | 简单 | 复杂 | 中等     |      |



