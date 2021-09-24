---
layout: post
title: "Run Debian iso on QEMU ARMv8"
description: "Debian ARM64 QEMU"
category: Tech
tags: [QEMU, embedded, Arm64, Debian]
---

##Run Debian iso on QEMU ARMv8

###Pre-installation

[download the debian iso](http://cdimage.debian.org/debian-cd/8.1.0/arm64/iso-cd/)

I used iso name is : debian-8.1.0-arm64-CD-1.iso

create img file and QEFI flash

```
$ dd if=/dev/zero of=flash0.img bs=1M count=64
$ LINARO_EDK2_URL=http://releases.linaro.org/15.01/components/kernel/uefi-linaro/
$ wget $LINARO_EDK2_URL/release/qemu64-intelbds/QEMU_EFI.fd
$ dd if=QEMU_EFI.fd of=flash0.img conv=notrunc
$ dd if=/dev/zero of=flash1.img bs=1M count=64
$ dd if=/dev/zero of=hda.img bs=1M count=8192
```

------------             
###lanuch.sh:

```
#!/bin/sh

CDROM_IMG=debian-8.1.0-arm64-CD-1.iso
HDA_IMG=hda.img

make_cdrom_arg()
{
  echo "-drive file=$1,id=cdrom,if=none,media=cdrom" \
    "-device virtio-scsi-device -device scsi-cd,drive=cdrom"
}

make_hda_arg()
{
  echo "-drive if=none,file=$1,id=hd0" \
    "-device virtio-blk-device,drive=hd0"
}

HDA_ARGS=`make_hda_arg $HDA_IMG`
if [ $# -eq 1 ]; then
  case $1 in
    install)
      CDROM_ARGS=`make_cdrom_arg $CDROM_IMG`
      ;;
    *)
      CDROM_ARGS=""
      ;;
  esac
fi

qemu-system-aarch64 -m 1024 -cpu cortex-a57 -M virt -nographic \
  -pflash flash0.img \
  $CDROM_ARGS $HDA_ARGS -netdev user,id=eth0 \
  -device virtio-net-device,netdev=eth0 
```

------------
execute ./lanuch.sh install

Then will show you install screen

###after install
```
sudo modprobe nbd max_part=63
sudo qemu-nbd -c /dev/nbd0 hda.img
mkdir mnt
sudo mount /dev/nbd0p2 mnt #Your rootfs partition, you can have a try nbd0p1~pN


sudo cp mnt/boot/vmlinuz-3.13.0-53-generic .
sudo cp mnt/boot/initrd.img-3.13.0-53-generic .


sudo umount mnt
sudo qemu-nbd -d /dev/nbd0
rmdir mnt
```

###trouble shotting
`nbd.c:nbd_init():L723: Failed to set NBD socket`

`ps -ef | grep "qemu"`

kill qemu-nbd process and retry

------

`qemu: fatal: Trying to execute code outside RAM or ROM at 0xffffffc000080000`

there is something wrong with the kernel, do not use `vmlinux`, use `vmlinuz` and check the kernel version

###run debian on QEMU

```
qemu-system-aarch64 -machine virt -cpu cortex-a57 -nographic -smp 1 -m 2048 \
	-pflash flash0.img \
  -drive if=none,file=hda.img,id=hd0 \
  -device virtio-blk-device,drive=hd0 \
	-kernel vmlinuz-3.16.0-4-arm64 \
	-initrd initrd.img-3.16.0-4-arm64 \
	-netdev user,id=unet -device virtio-net-device,netdev=unet \
	--append "console=ttyAMA0 root=/dev/vda2"
```

###how to share files between QEMU and host

There is a ponderous way to share files between QEMU and host:

create a `img` for share files.

```
dd if=/dev/zero of=share.img bs=1M count=1024
mkfs.ext4 share.img
mkdir mnt
mount -o loop share.img mnt
```

add the img file on the command to boot the QEMU

```
qemu-system-aarch64 -machine virt -cpu cortex-a57 -nographic -smp 1 -m 2048 \
	-pflash flash0.img \
	-drive file=debian-8.1.0-arm64-CD-1.iso,id=cdrom,if=none,media=cdrom \
	-device virtio-scsi-device -device scsi-cd,drive=cdrom \
  -drive if=none,file=share.img,id=hd1 \
  -device virtio-blk-device,drive=hd1 \
  -drive if=none,file=hda.img,id=hd0 \
  -device virtio-blk-device,drive=hd0 \
	-kernel vmlinuz-3.16.0-4-arm64 \
	-initrd initrd.img-3.16.0-4-arm64 \
	-netdev user,id=unet -device virtio-net-device,netdev=unet \
	--append "console=ttyAMA0 root=/dev/vda2"
```

###reference

http://blog.eciton.net/uefi/qemu-aarch64-jessie.html