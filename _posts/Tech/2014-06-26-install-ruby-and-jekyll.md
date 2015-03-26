---
layout: post
title: "install ruby and jekyll"
description: "install jekyll"
category: Tech
tags: [jekyll]
---


###使用淘宝gem镜像
参见：https://ruby.taobao.org/

###安装jekyll后使用 jekyll serve命令启动 默认访问路径 localhost:4000
使用markdown语言 后缀名为.md

###乱码错误解决
用编辑器修改编码方式为UTF8无BOM

###Windows安装jekyll
http://cn.yizeng.me/2013/05/10/setup-jekyll-on-windows/

###Windows下配置tortoiseGit与github
生成ssh-key和配置github上的ssh-key

使用tortoiseGit Push时，需要配置ssh-key

1. 如果之前有生成过ssh-key

    打开tortoiseGit/bin/puttygen.exe 点Load选择private key，之后再点save private key 保存为.ppk格式，再在push界面配置putty选项，选择刚才保存的.ppk即可

2. 之前没生成过(.ssh/下没有id_rsa)

    打开tortoiseGit/bin/puttygen.exe 点Generate。之后保存，再去github上配置ssh-key