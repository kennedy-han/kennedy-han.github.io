---
layout: post
title: "Linux ssh 省略密码登陆"
description: "Linux"
category: Tech
tags: [Linux, ssh]
---

### 场景
在自己本机想要快速ssh到（测试）服务器

#### 1. 生成公钥

```
ssh-keygen -t rsa -P ''
```

注：windows系统需要去掉-P参数，回车默认生成不带密码的key

#### 2. 将公钥拷贝到服务器

```
ssh-copy-id -i ~/.ssh/id_rsa.pub xxx@10.x.x.x
```

注：如果使用腾讯云等产品，或其他不知道ssh password时，但可以通过跳板机等方式登录的，可以手工将公钥添加进服务器 .ssh/authorized_keys 文件中

#### 3. 便于登陆，配置ssh config
example:

```
cat ~/.ssh/config

Host xxxName
  HostName  xxx.xxx.xxx.xxx
  User  root
```

#### 4. 访问ssh

```
ssh xxxName
```

