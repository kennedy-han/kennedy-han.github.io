---
layout: post
title: "Debian system initialization"
description: "Debian initialization"
category: Tech
tags: [Debian]
---

#Chapter 3. The system initialization

##3.1. An overview of the boot strap process

The typical boot strap process is like a four-stage rocket. Each stage rocket hands over the system control to the next stage one.

Section 3.1.1, “Stage 1: the BIOS”

Section 3.1.2, “Stage 2: the boot loader”

Section 3.1.3, “Stage 3: the mini-Debian system”

Section 3.1.4, “Stage 4: the normal Debian system”

Of course, these can be configured differently. For example, if you compiled your own kernel, you may be skipping the step with the mini-Debian system. So please do not assume this is the case for your system until you check it yourself.

###3.1.1. Stage 1: the BIOS
The BIOS is the 1st stage of the boot process which is started by the power-on event. The BIOS residing on the read only memory (ROM) is executed from the particular memory address to which the program counter of CPU is initialized by the power-on event.

The hardware location and the priority of the code started by the BIOS can be selected from the BIOS setup screen. Typically, the first few sectors of the first found selected device (hard disk, floppy disk, CD-ROM, …) are loaded to the memory and this initial code is executed. This initial code can be any one of the following.

The boot loader code

The kernel code of the stepping stone OS such as FreeDOS

The kernel code of the target OS if it fits in this small space

Typically, the system is booted from the specified partition of the primary hard disk partition. First 2 sectors of the hard disk on legacy PC contain the master boot record (MBR). The disk partition information including the boot selection is recorded at the end of this MBR. The first boot loader code executed from the BIOS occupies the rest of this MBR.

###3.1.2. Stage 2: the boot loader
The boot loader is the 2nd stage of the boot process which is started by the BIOS. It loads the system kernel image and the initrd image to the memory and hands control over to them. This initrd image is the root filesystem image and its support depends on the bootloader used.

The Debian system normally uses the Linux kernel as the default system kernel. The initrd image for the current 2.6/3.x Linux kernel is technically the `initramfs` (initial RAM filesystem) image. The initramfs image is a `gzipped` `cpio` archive of files in the root filesystem.

The default install of the Debian system places first-stage `GRUB` boot loader code into the MBR for the `PC` platform. There are many boot loaders and configuration options available.

For GRUB 2, the menu configuration file is located at `/boot/grub/grub.cfg`. It is automatically generated by `/usr/sbin/update-grub` using templates from `/etc/grub.d/*` and settings from `/etc/default/grub`. For example, it has entries as the following.

```
menuentry `Debian GNU/Linux` {
        set root=(hd0,3)
        linux /vmlinuz root=/dev/hda3
        initrd /initrd.img
}
```

For these examples, these GRUB parameters mean the following.

Table 3.2. The meaning of GRUB parameters

|GRUB parameter |	meaning |
|--------|
| root |	use 3rd partition on the primary disk by setting it as `(hd0,2)` in GRUB legacy or as `(hd0,3)` in GRUB 2 |
| kernel |	use kernel located at `/vmlinuz` with kernel parameter: `root=/dev/hda3 ro` |
| initrd |	use initrd/initramfs image located at `/initrd.img` |


The value of the partition number used by GRUB legacy program is one less than normal one used by Linux kernel and utility tools. GRUB 2 program fixes this problem.

If GRUB is used, the kernel boot parameter is set in `/boot/grub/grub.cfg`. On Debian system, you should not edit `/boot/grub/grub.cfg` directly. You should edit the GRUB_CMDLINE_LINUX_DEFAULT value in `/etc/default/grub` and run `update-grub(8)` to update `/boot/grub/grub.cfg`.

------

###3.1.3. Stage 3: the mini-Debian system
The mini-Debian system is the 3rd stage of the boot process which is started by the boot loader. It runs the system kernel with its root filesystem on the memory. This is an optional preparatory stage of the boot process.

The `/init` script is executed as the first program in this root filesystem on the memory. It is a shell script program which initializes the kernel in user space and hands control over to the next stage. This mini-Debian system offers flexibility to the boot process such as adding kernel modules before the main boot process or mounting the root filesystem as an encrypted one.

You can interrupt this part of the boot process to gain root shell by providing `break=init` etc. to the kernel boot parameter. See the `/init` script for more break conditions. This shell environment is sophisticated enough to make a good inspection of your machine's hardware.

Commands available in this mini-Debian system are stripped down ones and mainly provided by a GNU tool called busybox(1).

You need to use `-n` option for mount command when you are on the readonly root filesystem.

###3.1.4. Stage 4: the normal Debian system
The normal Debian system is the 4th stage of the boot process which is started by the mini-Debian system.The system kernel for the mini-Debian system continues to run in this environment. The root filesystem is switched from the one on the memory to the one on the real hard disk filesystem.

The init program is executed as the first program with `PID=1` to perform the main boot process of starting many programs. The default file path for the init program is `/sbin/init` but it can be changed by the kernel boot parameter as `init=/path/to/init_program`.

The default init program has been changing:

* Debian before squeeze uses the simple SysV-style init.

* Debian wheezy improves the SysV-style init by ordering the boot sequence with LSB header and starting boot scripts in parallel.

* Debian jessie switches its default init to the systemd for the event-driven and parallel initialization.

All boot mechanisms are compatible through `/etc/init.d/rc`, `/etc/init.d/rcS`, `/usr/sbin/update-rc.d`, and `/usr/sbin/invoke-rc.d` scripts.

The actual init command on your system can be verified by the `ps --pid 1 -f` command.

##3.2. SysV-style init
This section describes how the good old SysV-style init used to boot the system. Your Debian system does not function exactly as described here but it is quite educational to know this basics since the newer init system tends to offer equivalent functionalities.

The SysV-style boot process essentially goes through the following.

1. The Debian system goes into runlevel N (none) to initialize the system by following the `/etc/inittab` description.

2. The Debian system goes into runlevel S to initialize the system under the single-user mode to complete hardware initialization etc.

3. The Debian system goes into one of the specified multi-user runlevels (2 to 5) to start the system services.

The initial runlevel used for multi-user mode is specified with the `init=` kernel boot parameter or in the `initdefault` line of the `/etc/inittab`. The Debian system as installed starts at the runlevel 2.

All actual script files executed by the init system are located in the directory `/etc/init.d/`.

See init(8), inittab(5), and `/usr/share/doc/sysv-rc/README.runlevels.gz` for the exact explanation.

###3.2.1. The meaning of the runlevel
Each runlevel uses a directory for its configuration and has specific meaning as the following.

Table 3.4. List of runlevels and description of their usage

| runlevel |	directory |	description of runlevel usage |
|---|---|---|
|N |	none	|system bootup (NONE) level (no `/etc/rcN.d/` directory) |
|0 |	/etc/rc0.d/ |	halt the system |
|S |	/etc/rcS.d/ |	single-user mode on boot (alias: `s`) |
|1 |	/etc/rc1.d/ |	single-user mode switched from multi-user mode |
|2 |	/etc/rc2.d/ |	multi-user mode |
|3 |	/etc/rc3.d/ |	,, |
|4 |	/etc/rc4.d/ |	,, |
|5 |	/etc/rc5.d/ |	,, |
|6 |	/etc/rc6.d/ |	reboot the system |
|7 |	/etc/rc7.d/ |	valid multi-user mode but not normally used |
|8 |	/etc/rc8.d/ |	,, |
|9 |	/etc/rc9.d/ |	,, |

You can change the runlevel from the console to, e.g., 4 by the following.

```
$ sudo telinit 4
```

Caution

The Debian system does not pre-assign any special meaning differences among the runlevels between 2 and 5. The system administrator on the Debian system may change this. (I.e., Debian is not Red Hat Linux nor Solaris by Sun Microsystems nor HP-UX by Hewlett Packard nor AIX by IBM nor …)

The Debian system does not populate directories for the runlevels between 7 and 9 during installation. Traditional Unix variants don't use these runlevels.

###3.2.2. The configuration of the runlevel
When init(8) or telinit(8) commands goes into the runlevel to `<n>`, the system basically executes the initialization scripts as follows.

The script names starting with a `K` in `/etc/rc<n>.d/` are executed in alphabetical order with the single argument `stop`. (killing services)

The script names starting with an `S` in `/etc/rc<n>.d/` are executed in alphabetical order with the single argument `start`. (starting services)

For example, if you had the links `S10sysklogd` and `S20exim4` in a runlevel directory, `S10sysklogd` which is symlinked to `../init.d/sysklogd` would run before `S20exim4` which is symlinked to `../init.d/exim4`.

This simple sequential initialization system is the classical System V style boot system and was used up to the Debian lenny system.

The recent Debian system is optimized to execute the initialization scripts concurrently, instead.

###3.2.3. The runlevel management example

For example, let's set up runlevel system somewhat like Red Hat Linux as the following.

* init starts the system in runlevel=3 as the default.

* init does not start gdm3(1) in runlevel=(0,1,2,6).

* init starts gdm3(1) in runlevel=(3,4,5).

This can be done by using editor on the `/etc/inittab` file to change starting runlevel and using user friendly runlevel management tools such as sysv-rc-conf or bum to edit the runlevel. If you are to use command line only instead, here is how you do it (after the default installation of the gdm3 package and selecting it to be the choice of display manager).

```
# cd /etc/rc2.d ; mv S21gdm3 K21gdm3
# cd /etc ; perl -i -p -e 's/^id:.:/id:3:/' inittab
```

You can still start X from any console shell with the startx(1) command.

###3.2.4. The default parameter for each init script
The default parameter for each init script in `/etc/init.d/` is given by the corresponding file in `/etc/default/` which contains environment variable assignments only. This choice of directory name is specific to the Debian system. It is roughly the equivalent of the `/etc/sysconfig` directory found in Red Hat Linux and other distributions. For example, `/etc/default/cron` can be used to control how `/etc/init.d/cron` works.

The `/etc/default/rcS` file can be used to customize boot-time defaults for motd(5), sulogin(8), etc.

If you cannot get the behavior you want by changing such variables then you may modify the init scripts themselves. These are configuration files editable by system administrators.

###3.2.5. The hostname
The kernel maintains the system hostname. The init script in runlevel S which is symlinked to `/etc/init.d/hostname.sh` sets the system hostname at boot time (using the hostname command) to the name stored in `/etc/hostname`. This file should contain only the system hostname, not a fully qualified domain name.

To print out the current hostname run hostname(1) without an argument.

###3.2.6. The filesystem
Although the root filesystem is mounted by the kernel when it is started, other filesystems are mounted in the runlevel S by the following init scripts.

* `/etc/init.d/mountkernfs.sh` for kernel filesystems in `/proc`, `/sys`, etc.

* `/etc/init.d/mountdevsubfs.sh` for virtual filesystems in `/dev`

* `/etc/init.d/mountall.sh` for normal filesystems using `/etc/fstab`

* `/etc/init.d/mountnfs.sh` for network filesystems using`/etc/fstab`

The mount options of special kernel filesystems (procfs, sysfs, and tmpfs for /proc, /sys, /tmp, /run, etc.) are set in `/etc/default/rcS`. See rcS(5).

The mount options of normal disk and network filesystems are set in `/etc/fstab`. See Section 9.5.7, “Optimization of filesystem by mount options”.

After mounting all the filesystems, temporary files in `/tmp`, `/var/lock`, and `/var/run` are cleaned for each boot up.

###3.2.7. Network interface initialization
Network interfaces are initialized in runlevel S by the init script symlinked to `/etc/init.d/ifupdown-clean` and `/etc/init.d/ifupdown`. See Chapter 5, Network setup for how to configure them.

###3.2.8. Network service initialization
Many network services (see Chapter 6, Network applications) are started under multi-user mode directly as daemon processes at boot time by the init script, e.g., `/etc/rc2.d/S20exim4` (for RUNLEVEL=2) which is a symlink to `/etc/init.d/exim4`.

Some network services can be started on demand using the super-server inetd (or its equivalents). The inetd is started at boot time by `/etc/rc2.d/S20inetd` (for RUNLEVEL=2) which is a symlink to `/etc/init.d/inetd`. Essentially, inetd allows one running daemon to invoke several others, reducing load on the system.

Whenever a request for service arrives at super-server inetd , its protocol and service are identified by looking them up in the databases in `/etc/protocols` and `/etc/services`. inetd then looks up a normal Internet service in the `/etc/inetd.conf` database, or a Open Network Computing Remote Procedure Call (ONC RPC)/Sun RPC based service in `/etc/rpc.conf`.

###3.2.9. The system message
The system message can be customized by `/etc/default/rsyslog` and `/etc/rsyslog.conf` for both the log file and on-screen display. See rsyslogd(8) and rsyslog.conf(5). See also Section 9.2.2, “Log analyzer”.

###3.2.10. The kernel message
The kernel message can be customized by `/etc/default/klogd` for both the log file and on-screen display. Set `KLOGD='-c 3'` in this file and run `/etc/init.d/klogd restart`. See klogd(8).

You may directly change the error message level by the following.

```
# dmesg -n3
```

Table 3.5. List of kernel error levels

| error level value	| error level name |	meaning |
| ----------------- | ---------------- |----------| 
| 0									|KERN_EMERG   	| system is unusable |
| 1									|KERN_ALERT	    | action must be taken immediately |
| 2									|KERN_CRIT	    | critical conditions |
| 3									|KERN_ERR	    | error conditions |
| 4									|KERN_WARNING	| warning conditions |
| 5									|KERN_NOTICE	| normal but significant condition |
| 6									|KERN_INFO	    | informational |
| 7									|KERN_DEBUG	    | debug-level messages |


##3.3. The udev system
For Linux kernel 2.6 and newer, the udev system provides mechanism for the automatic hardware discovery and initialization (see udev(7)). Upon discovery of each device by the kernel, the udev system starts a user process which uses information from the sysfs filesystem (see Section 1.2.12, “procfs and sysfs”), loads required kernel modules supporting it using the modprobe(8) program (see Section 3.3.1, “The kernel module initialization”), and creates corresponding device nodes.

    * If `/lib/modules/<kernel-version>/modules.dep` was not generated properly by depmod(8) for some reason, modules may not be loaded as expected by the udev system. Execute `depmod -a` to fix it.
    
The name of device nodes can be configured by udev rule files in `/etc/udev/rules.d/`. Current default rules tend to create dynamically generated names resulting non-static device names except for cd and network devices. By adding your custom rules similar to what cd and network devices do, you can generate static device names for other devices such as USB memory sticks, too. See `Writing udev rules` or `/usr/share/doc/udev/writing_udev_rules/index.html`.

    * For mounting rules in `/etc/fstab`, device nodes do not need to be static ones. You can use UUID to mount devices instead of device names such as `/dev/sda`. See Section 9.5.3, “Accessing partition using UUID”.

###3.3.1. The kernel module initialization
The modprobe(8) program enables us to configure running Linux kernel from user process by adding and removing kernel modules. The udev system (see Section 3.3, “The udev system”) automates its invocation to help the kernel module initialization.

There are non-hardware modules and special hardware driver modules as the following which need to be pre-loaded by listing them in the `/etc/modules` file (see modules(5)).



The configuration files for the modprobe(8) program are located under the `/etc/modprobes.d/` directory as explained in modprobe.conf(5). (If you want to avoid some kernel modules to be auto-loaded, consider to blacklist them in the `/etc/modprobes.d/blacklist` file.)

The `/lib/modules/<version>/modules.dep` file generated by the depmod(8) program describes module dependencies used by the modprobe(8) program.

    * If you experience module loading issues with boot time module loading or with modprobe(8), `depmod -a` may resolve these issues by reconstructing `modules.dep`.
    
The modinfo(8) program shows information about a Linux kernel module.

The lsmod(8) program nicely formats the contents of the `/proc/modules`, showing what kernel modules are currently loaded.

------

This article is partly of the Official Document

https://www.debian.org/doc/manuals/debian-reference/ch03.en.html