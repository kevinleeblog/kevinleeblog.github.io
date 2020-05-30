---
layout: post
title: "Porting USB Over Ethernet to Android 7"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle: Android 7
visualworkflow: true
---

### 為何做?

有一個用來和Android連接透過USB的擴充板，這個擴充板上面的架構有一個USB Hub Chip，分成6個port連出去，其中兩個port是一般的USB Type A，另外一個Port是連接Microchip LAN7800是一個USB轉Ethernet的chip，其中兩個Port是連到USB轉RS232，最後一個Port是連接一顆MCU用來做GPIO的控制。
![無標題繪圖]({{site.baseurl}}/img/無標題繪圖.jpg)

而我的任務就是確認擴充板上的所有功能在公司自家的Android上動作正常，首先能夠確認的是USB Hub是正常因為USB Type A可以正常使用，但是USB LAN和USB Serial動作是不正常的。

### 如何做？

首先看USB LAN，在連接擴充板後，輸入ifconfig確認是沒有產生eth0 interface，但看網路孔指示燈是有亮的，所以懷疑是Linux Driver有些沒有編譯進去。

USB Over LAN使用Microchip LAN7800，查閱一下Linux Kernel，大約是Kernel version 4之後才有放進來
但公司其中一產品使用Android7其Kernel版本是3.18，顯然是沒有此chip的Driver

![image-20191225100950945]({{site.baseurl}}/img/image-20191225100950945.png)

> 如果是Android9之後的版本已經是Kernel 4了，所以就有包含Microchip LAN78xx的Driver了

所以想再Android-7上支援LAN78xx只剩下一條路，就是拿Kernel 4的backport到kernel 3.18
幸好！Microchip已經替我完成了，只要把patch file打到kernel 3.18
https://www.microchip.com/wwwproducts/en/LAN7800

![image-20191225101915876]({{site.baseurl}}/img/image-20191225101915876.png)

打完後，你以為就結束了嗎？還要把kernel config給設進去表示要使用LAN78xx的driver
因為我使用是Qualcomm的solution所以Kernel config要加在aosp的
`kernel/msm-3.18/arch/arm64/configs/msmcortex_defconfig`

```
CONFIG_MICROCHIP_PHY=y
CONFIG_USB_LAN78XX=y
```

這樣才會把Driver給編進來
在編譯的過程會報錯誤，放心不是你到錯，而是這driver寫的怪怪的
報錯的位址是lan78xx.c的1607行，錯誤的原因是cmd是const，但你卻傳給可能會更改到cmd的function內
把lan78xx_set_link_ksettings第二個參數的const給移掉就正常了

kernel/msm-3.18/drivers/net/usb/lan78xx.c

```
static int lan78xx_set_link_ksettings(struct net_device *net,
                                      const struct ethtool_cmd *cmd)
{
        /* change speed & duplex */
        ret = phy_ethtool_sset(phydev, cmd);
```

編譯好的image燒到裝置後，開機就抓得到LAN78xx及出現eth0 interface

![image-20191225104205022]({{site.baseurl}}/img/image-20191225104205022.png)

