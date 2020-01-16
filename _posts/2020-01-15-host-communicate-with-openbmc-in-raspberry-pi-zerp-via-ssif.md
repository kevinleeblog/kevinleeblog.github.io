---
layout: post
title: "Host OS communicate to raspberry pi with IPMI via SSIF "
auther: Kevin Lee
category: 
tags: [OpenBMC,Raspberry_Pi]
subtitle:
visualworkflow: true
---

### 為何做？

先前我已經成功把OpenBMC安裝於Raspberry Pi Zero W上，也學習了OpenBMC如何透過DBus傳送IPMI Messaging，為了要更深入了解BMC，就是要實際架設伺服器，這邊我和公司借了一塊伺服器主機板，上面使用的Chipset為伺服器等級的Intel C246

![image-20200115105440391]({{site.baseurl}}/img/image-20200115105440391.png)

第一件想做的事情就是讓Host OS可以與BMC(Paspberry Pi)直接溝通，因為沒有LPC，所以我想透過SSIF變成BMC與Host Host溝通的橋樑，這段網路上的資料比較少，所以我也要努力Study IPMI關於SSIF這段與C246的spec

目前OpenBMC尚未實現SSIF這段，所以這段要由我DIY去實現

### 如何做？

SSIF主要是透過SMBus，所以第一步就是先載入Raspberry Pi Zero W的i2c-dev module

```
$ modprobe i2c-dev
```

但是每次重開機就要重設一次
永久生效

```
$ vi /etc/modules-load.d/i2c-dev.conf
i2c-dev
```

重新開機就好

我這邊用Raspberry Pi Zero W，只有看到一個i2c設備

```
root@raspberrypi0-wifi:~# ls /dev/i2c*  
/dev/i2c-2
```

