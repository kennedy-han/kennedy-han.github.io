---
layout: post
title: "manage multi id_rsa"
description: "ssh id_rsa"
category: Tech
tags: [SSH, id_rsa]
---

##situation
You had multi ssh private key for Github or your company Git server.

##solution

create new key

```
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github -C "email"
```

run ssh-agent process

```
eval `ssh-agent -s`
```

add key 

```
$ ssh-add ~/.ssh/id_rsa
$ ssh-add ~/.ssh/id_rsa.github
```

create a `config` file in `.ssh` directory

```
touch ~/.ssh/config
```

config file contents:

```
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa.github
    
# osc
Host git.oschina.net
    HostName git.oschina.net
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa.osc   
```

HostName : is Git site domain
IdentityFile : is the path for private key

test:

```
ssh -T git@github.com
```