---
layout: post
title: "���豸����"
description: "���豸 ����"
category: Tech
tags: [embedded]
---


###���豸��������

�����ַ��豸��

С��������˯����������֪��С���Ƿ�˯��

1. ��ѯ��ʽ����ͣ�Ŀ��Ų鿴��̫��

2. ����/���ѣ�����Ҳ��������ȥ��Ϣ��С��˯�Ѻ�������

3. poll���ƣ���һ�����Ӵ���ʱʱ�䣬���С��һֱ˯�����ӳ�ʱ��ͻ᷵��

4. �첽֪ͨ�����źţ���С��˯�Ѻ󣬿���֪ͨ����

    ����4�㶼��ʹ���Լ��Ĵ��룬��ͨ��

5. ������ϵͳ��������˵Ĵ��루���˰����������1��2��3��4��

------

###���豸������ܣ�

```

app: open,read,write "1.txt"
----------------------------------------�ļ��Ķ�д
�ļ�ϵͳ��vfat,ext2,ext3,yaffs2,jffs2   (���ļ��Ķ�дת��Ϊ�����Ķ�д)
----------------ll_rw_block-------------�����Ķ�д
                                    1.��"��д"�������
                                    2.���ö��еĴ�����(�Ż�/��˳��/�ϲ�)
               ���豸��������
----------------------------------------
Ӳ����Ӳ��,flash

```

����μ���Linux�ں�Դ�����龰������

---

####����ll_rw_block��

```
          for (i = 0; i < nr; i++) {
                    struct buffer_head *bh = bhs[i];
                              submit_bh(WRITE, bh);
                                        struct bio *bio;     //ʹ��bh������bio (block input/output)
                                        submit_bio(rw, bio);
                                                  //ͨ�õĹ�������ʹ��bio����������(request)
                                                  generic_make_request(bio);
                                                            __generic_make_request(bio);
                                                                      request_queue_t *q = bdev_get_queue(bio->bi_bdev);//�ҵ�����
                                                                     
                                                                      //���ö��е�"����������"
                                                                      ret = q->make_request_fn(q, bio);
                                                                                //Ĭ�ϵĺ�����__make_request
                                                                                __make_request
                                                                                          //�ȳ��Ժϲ�(�����㷨)
                                                                                          elv_merge(q, &req, bio);
                                                                                         
                                                                                          //����ϲ����ɣ�ʹ��bio��������
                                                                                          init_request_from_bio(req, bio);
                                                                                         
                                                                                          //������������
                                                                                          add_request(q, req);
                                                                                         
                                                                                          //ִ�ж���
                                                                                          __generic_unplug_device(q);
                                                                                         
                                                                                          //���ö��е�"������"
                                                                                          q->request_fn(q);
                                                                                         
```

###��ôд���豸���������أ�
1. ����gendisk��alloc_disk
2. ���� 
 
    2.1 ����/���ö��У�request_queue_t     //���ṩ��д����
          blk_init_queue
          
    2.2 ����gendisk������Ϣ             //���ṩ���ԣ���������
3. ע�᣺add_disk

####�ο���
drivers/block/xd.c

drivers/block/z2ram.c


---

```
ls /dev/sd* -l
```

���豸��Ϊ0ʱ����ʾ�������̡�Ϊ1��2ʱ�����������
       
---                 
����3th:

�ڿ������ϣ�

1. insmod ramblock.ko
2. ��ʽ��:          mkdosfs /dev/ramblock
3. �ҽ�:               mount /dev/ramblock /tmp
4. ��д�ļ�: cd tmp     ����������ļ�
5. cd / && umount /tmp
6. cat /dev/ramblock > /mnt/ramblock.bin
7. ��PC�ϲ鿴ramblock.bin
          `sudo mount -o loop ramblock.bin /mnt      (loop ��һ����ͨ�ļ��������豸���ҽ�)`


####д�벻������д��Ҫ��һ��Ż�д

```
# cp /etc/inittab /tmp/
do_ramblock_requset read 43
#
# do_ramblock_requset write 6
do_ramblock_requset write 7
do_ramblock_requset write 8
do_ramblock_requset write 9


д�벻������д��һͬ��sync��������д
# cp /etc/init.d/rcS /tmp/
# sync
do_ramblock_requset write 10
do_ramblock_requset write 11
do_ramblock_requset write 12
do_ramblock_requset write 13
do_ramblock_requset write 14


д�벻������д��umountʱ�����д��
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

ʵ�飺ʹ���ڴ�ģ��Ӳ�̣�fdisk�Ϲ��ߣ�֧�ִ�ͷ������������getgeoģ����Щ

����5th:

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

fdisk�������2���������ֱ�����ʽ��

```
mkdosfs /dev/ramblock1
mkdosfs /dev/ramblock2
```

�ֱ����2������

```
mount /dev/ramblock1 /mnt
mount /dev/ramblock2 /tmp
```

ʹ��2��������д...