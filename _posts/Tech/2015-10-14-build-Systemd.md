---
layout: post
title: "build Systemd"
description: "Systemd"
category: Tech
tags: [Systemd]
---

#What is Systemd?
systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system.

for more see: http://www.freedesktop.org/wiki/Software/systemd/

```
/etc/sysctl.d/*.conf: a drop-in directory for kernel sysctl parameters, extending what you can already do with /etc/sysctl.conf.
```

See system boot up service sequence

`man bootup`

###build Systemd
get source

https://github.com/systemd/systemd

REQUIREMENTS

https://github.com/systemd/systemd/blob/master/README

```
./autogen.sh
./configure --prefix=/usr/local
make
(make install)
```

###Configuring kernel cmdline
To test systemd before switching to it by default, you can add the following boot parameter to the kernel:


`init=/bin/systemd`

This can be done in the grub menu for a single boot - press "e" in the grub menu and add this to the kernel line. For example, depending on the options required for your particular system, it might look something like:

```
linux   /vmlinuz-3.13-1-amd64 root=/dev/mapper/root-root init=/bin/systemd ro quiet
```

###troubleshooting
Error: `./autogen.sh: 31: ./autogen.sh: intltoolize: not found`

read REQUIREMENTS

and run `apt-get install intltool`

------

when `./configure `

Error: `libmount support required but libraries not found`

obtain util-linux

`git clone https://github.com/karelzak/util-linux.git`

```
cd util-linux
./autogen.sh
./configure --prefix=/usr/local
make
sudo make install
```

and then check the libmount version:

`cat /usr/local/include/libmount/libmount.h | grep "VERSION"`

make sure the version is higher than 2.27 that systemd requires.

------

Error: 

```
ValueError: failed to process ./man/busctl.xml
make[2]: *** [man/systemd.directives.xml] Error 1
make[2]: *** Deleting file `man/systemd.directives.xml'
make[1]: *** [all-recursive] Error 1
make: *** [all] Error 2
```

run 

`sudo apt-get install xsltproc`

`sudo apt-get install docbook-xsl`

and re-try.

------

###extra
How to check Kernel command line parameters/options?

```
# cat /proc/cmdline
```

###systemd-nspawn
systemd-nspawn is part of Systemd

Spawn a namespace container for debugging, testing and building

The systemd project needed a way to run inside of containers or virtual machines, but wanted a simple tool that was more like chroot than either LXC or libvirt LXC. Enter systemd-nspawn.

It is targeted at "building, testing, debugging, and profiling", not at deployment. 

---
####Spawn a shell in a container of a minimal Debian unstable distribution

```
# debootstrap --arch=amd64 unstable ~/debian-tree/ http://mirrors.163.com/debian/
# systemd-nspawn -D ~/debian-tree/
```

This installs a minimal Debian unstable distribution into the directory ~/debian-tree/ and then spawns a shell in a namespace container in it.

then set password for `root` account

```
passwd root
ctrl+] 3 times
```

setting network:

```
cd ~/debian-tree/
vi etc/resolv.conf
    nameserver 8.8.8.8
```

using this command to boot OS in container (using default network same with host)

```
# systemd-nspawn -bD ~/debian-tree/
```

login with `root` account which created upward

---


https://lwn.net/Articles/572957/

###systemd-networkd
Config the network using `systemd-networkd`

```
$ systemctl enable systemd-resolved
$ systemctl start systemd-resolved
$ systemctl enable systemd-networkd
$ systemctl start systemd-networkd
$ rm /etc/resolv.conf
$ ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
$ sudo mkdir /etc/systemd/network
```

####using static IP

```
$ sudo vi /etc/systemd/network/50-static.network

[Match]
Name=eth0

[Network]
Address=192.168.159.150/24
Gateway=192.168.159.255
```

####for more network config
http://www.freedesktop.org/software/systemd/man/systemd.network.html
https://linux.cn/article-6629-1.html

####compatibility with SysVinit
For this reason, we should delete eth0 in `/etc/network/interfaces` in order to use systemd-networkd, otherwise it will using `/etc/network/interfaces` configure eth0

```
cat /etc/network/interfaces
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
#allow-hotplug eth0
#iface eth0 inet dhcp
#------
#auto eth0
#iface eth0 inet static
#    address 192.168.159.159
#    netmask 255.255.255.0
#    gateway 192.168.159.255
```

####FAQ
Q: Why my change the network config and it doesn't change of command `ifconfig` or `ip a`

A: try `systemctl restart systemd-networkd` and `systemctl restart systemd-resolved`, finally use `reboot`

------

Q: Both `/etc/network/interfaces` and `/etc/systemd/network/50-static.network` configured the eth0, which config will be used ?

A: `/etc/network/interfaces` will recover the `/etc/systemd/network/50-static.network`

###Reference

http://www.freedesktop.org/software/systemd/man/bootup.html

http://www.linuxfromscratch.org/lfs/view/systemd/index.html

https://wiki.debian.org/systemd