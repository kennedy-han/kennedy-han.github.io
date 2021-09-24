---
layout: post
title: "QEMU ARM64 guide"
description: "QEMU ARM64 guide"
category: Tech
tags: [QEMU, embedded, Arm64]
---

##simulate a ARM64 environment, and run kernel on it

###Steps:

1. Build Dependancies

    ```
    sudo apt-get build-dep qemu
    ```

2. Building qemu

    ```
    git clone git://git.qemu.org/qemu.git qemu.git
    cd qemu.git
    ./configure –target-list=aarch64-softmmu
    make
    ```

3. Building your own rootfs

    ```
    git clone git://git.buildroot.net/buildroot buildroot.git
    cd buildroot.git
    make menuconfig
    ```

4. Building a kernel

    ```
    git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux.git
    cd linux.git
    ARCH=arm64 make menuconfig
    ARCH=arm64 make -j 8
    ```

5. Following Configurations are enabled in .config

    ```
    CONFIG_CROSS_COMPILE=”aarch64-linux-gnu-” # needs to match your cross-compiler prefix
    CONFIG_INITRAMFS_SOURCE=”/home/kennedy/dev/buildroot/output/images/rootfs.cpio” # points at your buildroot image
    CONFIG_NET_9P=y # needed for virtfs mount
    CONFIG_NET_9P_VIRTIO=y
    ```

For cross compilation i’m using gcc-linaro-aarch64-linux-gnu-4.9-2014.09_linux.tar.xz tool chain.

visit my blog [install linaro compiler](http://kennedy-han.github.io/2015/06/12/cross-compiler-install.html)

------

###run the kernel from .img

```
qemu-system-aarch64 -machine virt -cpu cortex-a57 -machine type=virt -nographic -smp 1 -m 2048 -kernel aarch64-linux-3.15rc2-buildroot.img  --append "console=ttyAMA0"
```

###run kernel Image

```
qemu-system-aarch64 -machine virt -cpu cortex-a57 -machine type=virt -nographic -smp 1 -m 2048 -kernel /home/kennedy/dev/kernel/linux-3.19.8/arch/arm64/boot/Image  --append "console=ttyAMA0"
```

------

###debugging kernel

```
qemu-system-aarch64 -s -S -machine virt -cpu cortex-a57 -machine type=virt -nographic -smp 1 -m 2048 -kernel /home/kennedy/dev/kernel/linux-3.19.8/arch/arm64/boot/Image  --append "console=ttyAMA0"
```

```
$cd linux-3.19.3
$aarch64-linux-gnu-gdb
$file vmlinux
$target remote localhost:1234
$b start_kernel
$c
$n 
```

###qemu-img

show the file info(format, size, etc.)

```
kennedy@kennedy-virtual-machine  ~/arm64  qemu-img info ubuntu-14.04-server-cloudimg-arm64-disk1.img
image: ubuntu-14.04-server-cloudimg-arm64-disk1.img
file format: qcow2
virtual size: 2.2G (2361393152 bytes)
disk size: 311M
cluster_size: 65536
Format specific information:
    compat: 0.10
    refcount bits: 16
```

###qemu-nbd

mount img file on filesystem

```
sudo modprobe nbd max_part=63
sudo qemu-nbd -c /dev/nbd0 ubuntu-14.04-server-cloudimg-arm64-disk1.img
mkdir mnt
sudo mount /dev/nbd0p1 mnt
```

search and copy files

```
sudo cp mnt/boot/vmlinuz-3.13.0-53-generic .
sudo cp mnt/boot/initrd.img-3.13.0-53-generic .
```

umount

```
sudo umount mnt
sudo qemu-nbd -d /dev/nbd0
rmdir mnt
```

####reference

http://www.bennee.com/~alex/blog/2014/05/09/running-linux-in-qemus-aarch64-system-emulation-mode/

http://blog.csdn.net/jefbai/article/details/44901447