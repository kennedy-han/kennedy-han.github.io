---
layout: post
title: "RTC驱动分析"
description: "RTC 驱动 分析"
category: Tech
tags: [embedded]
---

###RTC驱动分析

####断电后，还会记录时间，使用主板上电池用电量很低

```
drivers/rtc/rtc-s3c.c

s3c_rtc_init
          platform_driver_register
                    s3c_rtc_probe
                              rtc_device_register("s3c", &pdev->dev, &s3c_rtcops,THIS_MODULE);
                                        rtc_dev_prepare(rtc);
                                                  cdev_init(&rtc->char_dev, &rtc_dev_fops);
                                        rtc_dev_add_device(rtc);
                                                  cdev_add
                                                 
```

---

```                                                 
app:           open("/dev/rtc0");
----------------------------------------------
kernel:sys_open
                         rtc_dev_fops.open
                                   rtc_dev_open
                                             //根据次设备号找到以前用"rtc_device_register"注册的rtc_device
                                             struct rtc_device *rtc = container_of(inode->i_cdev,struct rtc_device, char_dev);
                                             const struct rtc_class_ops *ops = rtc->ops;
                                             err = ops->open ? ops->open(rtc->dev.parent) : 0;
                                                                                                                   s3c_rtc_open
```

---

```                                            
app:          ioctl(fd, RTC_RD_TIME,...)
----------------------------------------------
kernel:     sys_ioctl
                              rtc_dev_fops.ioctl
                                        rtc_dev_ioctl
                                                  struct rtc_device *rtc = file->private_data;
                                                  rtc_read_time(rtc, &tm);
                                                            err = rtc->ops->read_time(rtc->dev.parent, tm);
                                                                                          s3c_rtc_gettime
```

---

测试RTC:

1. 修改arch/arm/plat-s3c24xx/common-smdk.c

    ```
    static struct platform_device __initdata *smdk_devs[] = {
         &s3c_device_nand,
         &smdk_led4,
         &smdk_led5,
         &smdk_led6,
         &smdk_led7,
    改为(在数组smdk_devs里加上s3c_device_rtc)：
    static struct platform_device __initdata *smdk_devs[] = {
         &s3c_device_nand,
         &smdk_led4,
         &smdk_led5,
         &smdk_led6,
         &smdk_led7,
         &s3c_device_rtc,
    ```
2. make uImage 使用新内核启动
nfs 30000000 10.0.0.104:/work/nfs_root/tmp/fs_mini_mdev/uImage_rtc;bootm 30000000                                            

3. `ls /dev/rtc* -l`

    ```
              date                 /* 显示系统时间 */
              date022815202015.00  /* 设置系统时间 date [MMDDhhmm[[CC]YY][.ss]] */
              hwclock -w          /* 把系统时间写入RTC */
    ```         

断电，重启，执行date
         
####date只是设置软件时间，需要使用hwclock -w写入RTC