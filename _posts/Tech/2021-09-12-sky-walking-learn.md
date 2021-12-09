---
layout: post
title: "skywalking监控入门"
description: "skywalking"
category: Tech
tags: [skywalking]
---




## 官网

http://skywalking.apache.org/

优点：无侵入代码，运行期字节码增强



### APM

Application Performance Management,应用性能管理



### 启动

```
cd apache-skywalking-apm-bin-es7
cd bin
./startup.sh
SkyWalking OAP started successfully!
SkyWalking Web Application started successfully!
```



### 服务接入探针

```
#!/bin/sh
# SkyWalking Agent配置
export SW_AGENT_NAME=boot-micrometer #Agent名字,一般使用`spring.application.name`
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800 #配置 Collector 地址。
export SW_AGENT_SPAN_LIMIT=2000 #配置链路的最大Span数量，默认为 300。
export JAVA_AGENT=-javaagent:/root/apache-skywalking-apm-bin/agent/skywalking-agent.jar
java $JAVA_AGENT -jar springcloudalibaba-0.0.1-SNAPSHOT.jar #jar启动
```



### 集成IDE 启动项目

```
-javaagent:/xxxPath/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dskywalking.agent.service_name=xxxProjectName -Dskywalking.collector.backend_service=localhost:11800
```



### 监控dashboard 仪表盘

dashboard：http://localhost:8080/

**数据收集端口：**

- Http默认端口 12800

- gRPC默认端口 11800



```
上报数据接口使用11800，这里容易配错
```



------

### 指标

### Global全局维度

**Services load**：服务每分钟请求数

**Slow Services**：慢响应服务，单位ms

**Un-Health services(Apdex)**:Apdex性能指标，1为满分。

- Apdex 一个由众多网络分析技术公司和测量工业组成的联盟组织，它们联合起来开发了“应用性能指数”即“Apdex”(Application Performance Index)，用一句话来概括，Apdex是用户对应用性能满意度的量化值
- http://www.apdex.org/

**Slow Endpoints**: 慢响应端点，单位ms

**Global Response Latency**：百分比响应延时，不同百分比的延时时间，单位ms

**Global Heatmap**：服务响应时间热力分布图，根据时间段内不同响应时间的数量显示颜色深度



### Service服务维度

**Service Apdex（数字**）:当前服务的评分 

**Service Avg Response Times**：平均响应延时，单位ms

**Successful Rate（数字）**：请求成功率

**Servce Load（数字）**：每分钟请求数

**Service Apdex（折线图）**：不同时间的Apdex评分

**Service Response Time Percentile**：百分比响应延时

**Successful Rate（折线图）**：不同时间的请求成功率

**Servce Load（折线图）**：不同时间的每分钟请求数

**Servce Instances Load**：每个服务实例的每分钟请求数

**Slow Service Instance**：每个服务实例的最大延时

**Service Instance Successful Rate**：每个服务实例的请求成功率



### Instance

**Service Instance Load**：当前实例的每分钟请求数

**Service Instance Successful Rate**：当前实例的请求成功率

**Service Instance Latency**：当前实例的响应延时

**JVM CPU**:jvm占用CPU的百分比

**JVM Memory**：JVM内存占用大小，单位m

**JVM GC Time**：JVM垃圾回收时间，包含YGC和OGC

**JVM GC Count**：JVM垃圾回收次数，包含YGC和OGC



### Endpoint

**Endpoint Load in Current Service**：每个端点的每分钟请求数

**Slow Endpoints in Current Service**：每个端点的最慢请求时间，单位ms

**Successful Rate in Current Service**：每个端点的请求成功率

**Endpoint Load**：当前端点每个时间段的请求数据

**Endpoint Avg Response Time**：当前端点每个时间段的请求行响应时间

**Endpoint Response Time Percentile**：当前端点每个时间段的响应时间占比

**Endpoint Successful Rate**：当前端点每个时间段的请求成功率

### 仪表盘空数据的情况
页面右上角，有个同步的圆圈图标（skywalking apm 8.9版本）或者下拉三角按钮
选中“自动” 默认6秒刷新


### 参考

官方示例

http://122.112.182.72:8080/



CAT、Zipkin和SkyWalking的优缺点

https://www.jianshu.com/p/9bb660190884

skywalking仪表盘无数据，查看日志log，可能是plugins没导入

https://blog.csdn.net/qq_43437874/article/details/108615005

在Agent日志中报错，相关插件错误，解决方法：将报错不使用的插件，从plugins目录，移动到optional-plugins目录
plugins目录为正在使用的插件
optional-plugins 顾名思义，是可选用的插件
https://blog.csdn.net/lizz861109/article/details/107567740
