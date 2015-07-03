---
layout: post
title: "Debug Assembly for ARMv8 on QEMU"
description: "Debug Assembly ARM QEMU"
category: Tech
tags: [QEMU, embedded, Arm64, Assembly, Debug, GDB]
---

##Debug Assembly with GDB for ARMv8 on QEMU

###build from source code

assume you have installed QEMU, cross-compiler include cross-platform GDB

ARMv8 source code:

`test.S`

```
.globl _start

_start:
  mov x0, #0
  mov x1, #1
  mov x2, #2
loop:
  b loop
```

* compile:

    `aarch64-linux-gnu-as -o test.o test.S`

* link:

    `aarch64-linux-gnu-ld -o test test.o -o test test.o`

* objdump (optional):

    `aarch64-linux-gnu-objdump -d test`

* start gdb server: (use port 11111 and run in deamon)

    `qemu-aarch64 -g 11111 test &`

* start gdb:    (specified the execute file)

    `aarch64-linux-gnu-gdb test`

###GDB commands

```
(gdb) target remote localhost:11111     #connect from localhost:11111
(gdb) disassemble           #show the disassemble
(gdb) display /10i $pc-16       #show 10 instruction near the PC register

(gdb) x /16 0x400000           #show 16 word on memory 0x400000
(gdb) b *0x400080     #add a breakpoint with address


(gdb) ni    #next instruction
(gdb) si    #step instruction
(gdb) c    #continue exec until breakpoint

(gdb) info register         #show register
(gdb) display  /x $x0   #display x0 register
```





