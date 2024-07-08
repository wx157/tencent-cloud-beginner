---
title: 云服务器开启虚拟内存 SWAP
description: 云服务器开启虚拟内存 SWAP
published: 1
date: 2024-07-08T08:18:21.391Z
tags: 
editor: markdown
dateCreated: 2024-07-08T08:17:14.601Z
---

# 云服务器开启虚拟内存 SWAP

云服务器一般默认禁止虚拟内存，但是经常会遇到运行一个占内存的程序导致云服务器失联的情况，究其原因，可能是因为内存不够，导致操作系统把ssh服务杀了，从而导致失联。

## 开启虚拟内存

可以使用如下脚本开启虚拟内存[^1]：

```shell
#! /bin/bash

# 创建虚拟内存文件
dd if=/dev/zero of=/mnt/swap bs=1M count=$((4*1024))
chmod 0600 /mnt/swap
mkswap /mnt/swap
swapon /mnt/swap
# 写入自动挂载参数
if ! grep -q swap /etc/fstab; then
    echo "/mnt/swap swap swap defaults 0 0" >> /etc/fstab
fi
# 设置虚拟内存使用率
if ! grep -q swappiness /etc/sysctl.conf; then
    echo "vm.swappiness = 10" >> /etc/sysctl.conf
else
    sed -i 's/vm.swappiness = 0/vm.swappiness = 10/' /etc/sysctl.conf
fi
# 使配置生效
sysctl -p
```

`bs=1M count=$((4*1024))` 表示创建一个4G（1M * 4 * 1024）大小的块文件。
`swappiness=0` 表示最大限度使用物理内存，然后才是swap空间。
`swappiness＝100` 表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。

## 关闭虚拟内存

可以使用如下脚本关闭虚拟内存：

```shell
#! /bin/bash

swapoff /mnt/swap
sed -i '\/mnt\/swap/d' /etc/fstab
rm -f /mnt/swap
```

[^1]:  参考自 [云服务器开启虚拟内存 SWAP](https://www.rehiy.com/post/427/)