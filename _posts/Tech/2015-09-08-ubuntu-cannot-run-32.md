---
layout: post
title: "ubuntu 64bits cannot run 32bits app"
description: "ubuntu 64bits cannot run 32bits app"
category: Tech
tags: [Ubuntu]
---

###Problems:
when you run a application, it shows 

```
********** No such file or directory
```

To solve this problem, use 

```
sudo apt-get install ia32-libs
```

There are little change since ubuntu 13.10.

After ubuntu 13.10, use

```
sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0
```