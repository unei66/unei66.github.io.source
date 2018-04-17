---
layout: post
title:  "Centos7内核添加hello world模块"
date:   2016-10-27 00:29
categories: Linux
---

## 下载源码
https://wiki.centos.org/zh/HowTos/I_need_the_Kernel_Source

## 简单demo代码

    ```
    #include <linux/init.h>
    #include <linux/module.h>
    #include <linux/kernel.h>
    //必选
    //模块许可声明
    MODULE_LICENSE("GPL");
    //模块加载函数
    static int hello_init(void)
    {
        printk(KERN_ALERT "hello,I am edsionte\n");
        return 0;
    }
    //模块卸载函数
    static void hello_exit(void)
    {
        printk(KERN_ALERT "goodbye,kernel\n");
    }
    //模块注册
    module_init(hello_init);
    module_exit(hello_exit);
    //可选
    MODULE_AUTHOR("edsionte Wu");
    MODULE_DESCRIPTION("This is a simple example!\n");
    MODULE_ALIAS("A simplest example");
    ```

代码来自：http://edsionte.com/techblog/archives/1336

## 编译模块

官方编译指南：
https://wiki.centos.org/zh/HowTos/BuildingKernelModules

上一下我的脚本(就是将官方的整合一下)：

```
    #!/bin/bash
    /home/unei/rpmbuild/BUILD/kernel-3.10.0-327.36.2.el7/linux-3.10.0-327.36.2.el7.x86_64
    make oldconfig
    make menuconfig
    make prepare
    make modules_prepare
    make M=/home/unei/rpmbuild/DIY_MODULES/helloworld
    cp /home/unei/rpmbuild/DIY_MODULES/helloworld/hello.ko /lib/modules/`uname -r`/extra
    depmod -a
```

编译并安装后可能会遇到一下问题

+ dmesg 可能会出现hello: no symbol version for module_layout
+ /var/log/messages可能也会出现类似信息

以上说明模块加载不成功

## 解决方法及验证命令

+ 强制安装：modprobe -f hello
+ 查看kernel模块信息：lsmod
+ 查看某个模块信息：modinfo hello

上述信息来自：
https://www.centos.org/docs/5/html/5.1/Deployment_Guide/s1-kernel-module-utils.html
