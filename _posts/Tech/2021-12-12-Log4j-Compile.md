---
layout: post
title: "Log4j Compile"
description: "Log4j"
category: Tech
tags: [Log4j, Compile]
---

### 原因：
log4j漏洞

漏洞链接参考：
https://www.infoq.cn/article/SOQt4DKdWUi652W2CmRh


### 补丁下载地址：
https://github.com/apache/logging-log4j2/releases/tag/log4j-2.15.0-rc2


#### 1. 下载源码

```
wget https://github.com/apache/logging-log4j2/archive/refs/tags/log4j-2.15.0-rc2.tar.gz
tar zxvf log4j-2.15.0-rc2.tar.gz
cd logging-log4j2-log4j-2.15.0-rc2
```

#### 2. 下载Maven

```
wget https://dlcdn.apache.org/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz
tar zxvf apache-maven-3.8.4-bin.tar.gz
mv apache-maven-3.8.4 /xxx
```

#### 3. 下载安装JDK 9

随着JDK版本日新月异，开源社区使用的版本也在更新，编译时需要用到JDK 9

从Oracle官网下载(已经放在了/root目录下 jdk-9.0.4_linux-x64_bin.tar.gz)
```
tar zxvf jdk-9.0.4_linux-x64_bin.tar.gz
mv jdk-9.0.4 /xxx
```

#### 4. 配置环境变量

注意，环境变量配置成JDK8

```
cd
vi .bashrc

export JAVA_HOME=/usr/java/jdk1.8.0_144
export PATH=$JAVA_HOME/bin:$PATH
export M2_HOME=/xxx/apache-maven-3.8.4
export PATH=$M2_HOME/bin:$PATH
```

5. 准备好toolchains

[什么是toolchains](https://zhuanlan.zhihu.com/p/142960106)

```
cd /root/logging-log4j2-log4j-2.15.0-rc2
vi toolchains-sample-linux.xml
```

配置好JDK8 和 JDK9版本路径
```
<?xml version="1.0" encoding="UTF8"?>
<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one or more
  ~ contributor license agreements. See the NOTICE file distributed with
  ~ this work for additional information regarding copyright ownership.
  ~ The ASF licenses this file to You under the Apache license, Version 2.0
  ~ (the "License"); you may not use this file except in compliance with
  ~ the License. You may obtain a copy of the License at
  ~
  ~      http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the license for the specific language governing permissions and
  ~ limitations under the license.
  -->
<toolchains>
  <!-- JDK toolchains -->
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>1.8</version>
      <vendor>sun</vendor>
    </provides>
    <configuration>
      <jdkHome>/usr/java/jdk1.8.0_144</jdkHome>
    </configuration>
  </toolchain>
  <toolchain>
    <type>jdk</type>
    <provides>
      <version>9</version>
      <vendor>sun</vendor>
    </provides>
    <configuration>
      <jdkHome>/xxx/jdk-9.0.4</jdkHome>
    </configuration>
  </toolchain>

  <!-- other toolchains -->
</toolchains>
```

6. 开始编译

阅读 README.md 得知使用 `mvn install` 编译

```
cd /root/logging-log4j2-log4j-2.15.0-rc2
```

如果在顶层目录直接执行 `mvn install`
可能会报错，log4j-api-java9 相关的错误

```
cd log4j-api-java9
mvn install -t ../toolchains-sample-linux.xml -Dmaven.test.skip=true
```

还有个目录 log4j-core-java9 也会报错，也需要进去编译

```
cd log4j-core-java9
mvn install -t ../toolchains-sample-linux.xml -Dmaven.test.skip=true
```

之后来到顶级目录继续编译
```
/root/logging-log4j2-log4j-2.15.0-rc2
mvn install -t toolchains-sample-linux.xml -Dmaven.test.skip=true
```

会得到报错：
log4j-perf 相关的错误，因为它需要 JDK 11来编译（设置toolchains)

在pom.xml 中注释掉不需要编译的模块组件(module)


```
vi pom.xml

注释掉
<module>log4j-perf</module>
<module>log4j-jpl</module>
```

继续编译即可
```
mvn install -t toolchains-sample-linux.xml -Dmaven.test.skip=true
```

这里跳过了单测，因为测试服务器占用了某些端口，查看单测执行失败的日志也显示端口占用，故跳过单测
编译大概10分钟左右
