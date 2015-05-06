---
layout: post
title: "Assembly 寄存器"
description: "Assembly reg"
category: Tech
tags: [Assembly]
---


###通用寄存器
8086CPU的所有寄存器都是16位，可以存放两个字节。AX、BX、CX、DX这4个寄存器通常用来存放一般性的数据，被称为通用寄存器

都可以分为2个独立的8位寄存器来用

AX可分为AH和AL

###字在寄存器中的存储
字节：记为byte 一个字节由8个bit组成

字：  记为word 由两个字节组成

------

字：01001110 00100000

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;高位字节&nbsp;&nbsp;&nbsp;低位字节

------

###汇编指令

|汇编指令   |控制CPU完成的操作         |用高级语言的语法描述|
|----------------------------------------
|mov ax,18 | 将18送入寄存器ax          |AX=18|
|mov ah,78 | 将78送入寄存器AH          |AH=78|
|add ax,8  | 将寄存器AX中的数值加上8    |AX=AX+8|
|mov ax,bx | 将寄存器BX中的值送入寄存器AX|AX=BX|
|add ax,bx | 将AX和BX中的数值相加，结果存在AX中|AX=AX+BX|

###8086CPU给出物理地址的方法
段地址x16+偏移地址=物理地址

###CS和IP
CS为代码段寄存器，IP为指令指针寄存器

在8086PC机种，`任意时刻`,设CS中的内容为M，IP中的内容为N，8086CPU将从内存Mx16+N单元开始，读取一条指令并执行。

读取一条指令后，IP的值会自动增加

###修改CS、IP的指令
jmp 段地址：偏移地址

jmp 某一合法寄存器

###8086CPU的工作过程
1.从CS:IP指向的内存单元读取指令，读取的指令进入指令缓冲器

2.IP指向下一条指令

3.执行指令。（转到步骤1，重复这个过程）

###debug 常用命令
查看、修改CPU寄存器的内容：R

查看内存：D

修改内存：E

将内存中的内容解释为机器指令和对应的汇编指令：U

执行CS:IP指向的内存单元处的指令：T

以汇编指令的形式向内存中写入指令：A