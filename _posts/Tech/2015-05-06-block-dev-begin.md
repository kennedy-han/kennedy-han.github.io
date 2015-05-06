---
layout: post
title: "块设备驱动"
description: "块设备 驱动"
category: Tech
tags: [embedded]
---


###块设备驱动程序

回忆字符设备：

小孩在屋里睡觉，妈妈想知道小孩是否睡醒

1. 查询方式，不停的开门查看，太累

2. 休眠/唤醒，妈妈也进屋子里去休息，小孩睡醒后唤醒妈妈

3. poll机制，加一个闹钟代表超时时间，如果小孩一直睡，闹钟超时后就会返回

4. 异步通知（发信号），小孩睡醒后，开门通知妈妈

    以上4点都是使用自己的代码，不通用

5. 输入子系统，融入别人的代码（别人帮我们完成了1、2、3、4）

------

###块设备驱动框架：

```

app: open,read,write "1.txt"
----------------------------------------文件的读写
文件系统：vfat,ext2,ext3,yaffs2,jffs2   (把文件的读写转换为扇区的读写)
----------------ll_rw_block-------------扇区的读写
                                    1.把"读写"放入队列
                                    2.调用队列的处理函数(优化/调顺序/合并)
               块设备驱动程序
----------------------------------------
硬件：硬盘,flash

```

详情参见《Linux内核源代码情景分析》

---

####分析ll_rw_block：

```
          for (i = 0; i < nr; i++) {
                    struct buffer_head *bh = bhs[i];
                              submit_bh(WRITE, bh);
                                        struct bio *bio;     //使用bh来构造bio (block input/output)
                                        submit_bio(rw, bio);
                                                  //通用的构造请求：使用bio来构造请求(request)
                                                  generic_make_request(bio);
                                                            __generic_make_request(bio);
                                                                      request_queue_t *q = bdev_get_queue(bio->bi_bdev);//找到队列
                                                                     
                                                                      //调用队列的"构造请求函数"
                                                                      ret = q->make_request_fn(q, bio);
                                                                                //默认的函数是__make_request
                                                                                __make_request
                                                                                          //先尝试合并(电梯算法)
                                                                                          elv_merge(q, &req, bio);
                                                                                         
                                                                                          //如果合并不成，使用bio构造请求
                                                                                          init_request_from_bio(req, bio);
                                                                                         
                                                                                          //把请求放入队列
                                                                                          add_request(q, req);
                                                                                         
                                                                                          //执行队列
                                                                                          __generic_unplug_device(q);
                                                                                         
                                                                                          //调用队列的"处理函数"
                                                                                          q->request_fn(q);
                                                                                         
```

###怎么写块设备驱动程序呢？
1. 分配gendisk：alloc_disk
2. 设置 
 
    2.1 分配/设置队列：request_queue_t     //它提供读写能力
          blk_init_queue
          
    2.2 设置gendisk其他信息             //它提供属性：比如容量
3. 注册：add_disk

####参考：
drivers/block/xd.c

drivers/block/z2ram.c


---

```
ls /dev/sd* -l
```

次设备号为0时，表示整个磁盘。为1、2时，代表分区号
       
---                 
测试3th:

在开发板上：

1. insmod ramblock.ko
2. 格式化:          mkdosfs /dev/ramblock
3. 挂接:               mount /dev/ramblock /tmp
4. 读写文件: cd tmp     在里面操作文件
5. cd / && umount /tmp
6. cat /dev/ramblock > /mnt/ramblock.bin
7. 在PC上查看ramblock.bin
          `sudo mount -o loop ramblock.bin /mnt      (loop 把一个普通文件当作块设备来挂接)`


####写入不会立即写，要等一会才会写

```
# cp /etc/inittab /tmp/
do_ramblock_requset read 43
#
# do_ramblock_requset write 6
do_ramblock_requset write 7
do_ramblock_requset write 8
do_ramblock_requset write 9


写入不会立即写，一同步sync，则立即写
# cp /etc/init.d/rcS /tmp/
# sync
do_ramblock_requset write 10
do_ramblock_requset write 11
do_ramblock_requset write 12
do_ramblock_requset write 13
do_ramblock_requset write 14


写入不会立即写，umount时会完成写入
# cp /mnt/ramblock.ko /tmp
# cd /
# umount /tmp
do_ramblock_requset write 15
do_ramblock_requset write 16
do_ramblock_requset write 17
do_ramblock_requset write 18
do_ramblock_requset write 19
do_ramblock_requset write 20
do_ramblock_requset write 21
do_ramblock_requset write 22
do_ramblock_requset write 23
do_ramblock_requset write 24
do_ramblock_requset write 25
do_ramblock_requset write 26
do_ramblock_requset write 27
do_ramblock_requset write 28
do_ramblock_requset write 29
do_ramblock_requset write 30
do_ramblock_requset write 31
do_ramblock_requset write 32
do_ramblock_requset write 33
do_ramblock_requset write 34
do_ramblock_requset write 35
do_ramblock_requset write 36
do_ramblock_requset write 37
do_ramblock_requset write 38
```

---

实验：使用内存模拟硬盘，fdisk老工具，支持磁头柱面扇区，用getgeo模拟这些

测试5th:

1. insmod ramblock.ko
2. ls /dev/ramblock* -l

```
# ls /dev/ramblock* -l
brw-rw----    1 0        0        254,   0 Jan  1 00:42 /dev/ramblock

3. fdisk /dev/ramblock
4. ls /dev/ramblock* -l

# ls /dev/ramblock* -l
brw-rw----    1 0        0        254,   0 Jan  1 00:34 /dev/ramblock
brw-rw----    1 0        0        254,   1 Jan  1 00:36 /dev/ramblock1
brw-rw----    1 0        0        254,   2 Jan  1 00:36 /dev/ramblock2
```

fdisk这里分了2个分区，分别对其格式化

```
mkdosfs /dev/ramblock1
mkdosfs /dev/ramblock2
```

分别挂载2个分区

```
mount /dev/ramblock1 /mnt
mount /dev/ramblock2 /tmp
```

使用2个分区读写...