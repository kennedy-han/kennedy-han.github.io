---
layout: post
title: "QEMU install"
description: "QEMU"
category: Tech
tags: [QEMU, embedded]
---

##what is QEMU ?
QEMU is a generic and open source machine emulator and virtualizer.[www.qemu.org](http://www.qemu.org)

###before install

```
sudo apt-get install libtool
```

###get QEMU and install

I use `aarch64`, so you had better change the `./configure` param that you need.

```
$ git clone git://git.qemu-project.org/qemu.git
 Cloning into 'qemu'...
 remote: Counting objects: 131834, done.
 remote: Compressing objects: 100% (29320/29320), done.
 remote: Total 131834 (delta 104345), reused 129302 (delta 102090)
 Receiving objects: 100% (131834/131834), 45.42 MiB | 300 KiB/s, done.
 Resolving deltas: 100% (104345/104345), done.
 Checking out files: 100% (2849/2849), done.
$ cd qemu/
$ ./configure --target-list=aarch64-softmmu
$ make
$ sudo make install
```

###run a test

download a `.img` file or made a `.img` file by yourself.

then run the qemu

```
wget http://people.linaro.org/~alex.bennee/images/aarch64-linux-3.15rc2-buildroot.img

./aarch64-softmmu/qemu-system-aarch64 -machine virt -cpu cortex-a57 -machine type=virt -nographic -smp 1 -m 2048 -kernel aarch64-linux-3.15rc2-buildroot.img  --append "console=ttyAMA0"
```

###exit from QEMU

`ctrl+a` + `c`

and type `quit`

