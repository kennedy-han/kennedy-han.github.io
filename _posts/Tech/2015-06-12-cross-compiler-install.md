---
layout: post
title: "cross compiler install"
description: "QEMU"
category: Tech
tags: [QEMU, embedded, Arm64]
---

###install Linaro's compiler
The gcc compiler for ARMv8(aarch64) architecture

first download the compiler file, visit Linaro site.

and then

```
#.tar.xz convert to .tar
xz -d gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux.tar.xz
```

```
#uncompress tar
tar xvf gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux.tar
```

```
#put it in /opt
sudo mv gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux /opt
cd /opt
sudo mv gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux aarch64-gcc
```

```
#build a soft link in /usr/bin
cd /usr/bin
sudo ln -s /opt/aarch64-gcc/bin/* .
```

####prepare environment

```
cd /lib
sudo mkdir aarch64-linux-gnu
cd /opt
sudo cp ./aarch64-gcc/aarch64-linux-gnu/libc/lib/ld-linux-aarch64.so.1 /lib/aarch64-linux-gnu
sudo cp ./aarch64-gcc/aarch64-linux-gnu/libc/lib/aarch64-linux-gnu/libc.so.6 /lib/aarch64-linux-gnu
sudo cp ./aarch64-gcc/aarch64-linux-gnu/lib/libgcc_s.so /lib/aarch64-linux-gnu

cd /lib
sudo ln -s aarch64-linux-gnu/ld-linux-aarch64.so.1 ld-linux-aarch64.so.1
```

####show version

```
aarch64-linux-gnu-gdb --version
```

####test helloworld

write a helloworld C file and then

```
aarch64-linux-gnu-gcc hello.c -o hello
```

if you have prepared environment on the top, then run

```
qemu-aarch64 hello
```

otherwise, you should add a link path and run

```
qemu-aarch64 -L /usr/aarch64-linux-gnu/ hello 
```

####or use `-static` option

```
aarch64-linux-gnu-gcc -static hello.c -o hello
qemu-aarch64 hello
```