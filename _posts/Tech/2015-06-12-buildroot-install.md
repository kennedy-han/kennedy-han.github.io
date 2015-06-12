---
layout: post
title: "buildroot install"
description: "buildroot"
category: Tech
tags: [buildroot, embedded, Arm64]
---

##buildroot install
buildroot is a tool set. It contains : bootloader,busybox,kernel etc.

[wiki](http://en.wikipedia.org/wiki/Buildroot)

###Pre-installation

```
sudo apt-get install libncurses-dev
```

###install

```
git clone git://git.buildroot.net/buildroot
cd buildroot
make menuconfig
```

* Target Options -> Target Architecture(AArch64)
* Toolchain -> Toolchain type (External toolchain)
* Toolchain -> Toolchain (Linaro AArch64 14.02)
* System configuration -> Run a getty (login prompt) after boot (BR2_TARGET_GENERIC_GETTY)
* System configuration -> getty options -> TTY Port (ttyAMA0) (BR2_TARGET_GENERIC_GETTY_PORT)
* Target Packages -> Show packages that are also provided by busybox (BR2_PACKAGE_BUSYBOX_SHOW_OTHERS)
* Filesystem images -> cpio the root filesystem (for use as an initial RAM filesystem) (BR2_TARGET_ROOTFS_CPIO)

```
make
```

*note: getty is a Unix program, it provides login function. [wiki](http://en.wikipedia.org/wiki/Getty_(Unix))*
