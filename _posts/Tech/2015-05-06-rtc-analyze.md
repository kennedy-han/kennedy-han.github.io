---
layout: post
title: "RTC��������"
description: "RTC ���� ����"
category: Tech
tags: [embedded]
---

###RTC��������

####�ϵ�󣬻����¼ʱ�䣬ʹ�������ϵ���õ����ܵ�

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
                                             //���ݴ��豸���ҵ���ǰ��"rtc_device_register"ע���rtc_device
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

����RTC:

1. �޸�arch/arm/plat-s3c24xx/common-smdk.c

    ```
    static struct platform_device __initdata *smdk_devs[] = {
         &s3c_device_nand,
         &smdk_led4,
         &smdk_led5,
         &smdk_led6,
         &smdk_led7,
    ��Ϊ(������smdk_devs�����s3c_device_rtc)��
    static struct platform_device __initdata *smdk_devs[] = {
         &s3c_device_nand,
         &smdk_led4,
         &smdk_led5,
         &smdk_led6,
         &smdk_led7,
         &s3c_device_rtc,
    ```
2. make uImage ʹ�����ں�����
nfs 30000000 10.0.0.104:/work/nfs_root/tmp/fs_mini_mdev/uImage_rtc;bootm 30000000                                            

3. `ls /dev/rtc* -l`

    ```
              date                 /* ��ʾϵͳʱ�� */
              date022815202015.00  /* ����ϵͳʱ�� date [MMDDhhmm[[CC]YY][.ss]] */
              hwclock -w          /* ��ϵͳʱ��д��RTC */
    ```         

�ϵ磬������ִ��date
         
####dateֻ���������ʱ�䣬��Ҫʹ��hwclock -wд��RTC