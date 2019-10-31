---
layout: post
title: "openSUSE/SUSE Network Configuration"
auther: Kevin Lee
category: 
tags: [Linux, Job_Logging]
subtitle:
visualworkflow: true
---

#### 為何做？

業務想要爭取客戶訂單，其中客戶要求Platform要支援Linux Suse12，所以QA就在Platform上去驗證周邊裝置的功能，其中發現網路eth0/eth1沒有顯示，只有lo出現
所以事情就分配到我這去找問題

#### 如何做？

QA特別和我說，開機log有出現eth0/eth1的訊息，但不知為何後來沒出現且網路孔的顯示燈也沒亮
使用*ifconfig*指令，確實只有出現lo Interface，但對於在OpenBMC 處理過NIC的我來說已經見怪不怪
就把所有網路介面給顯現吧

```
$ip a
```

果然看到eth0/eth1在一個DOWN的狀態，表示網路驅動至少是正常的

喚醒eth0/eth1

```
$ ip link set dev eth0 up
$ ip link set dev eth1 up
```

網路孔的指示燈亮起，輸入ifconfig
所有的周邊裝置都出現了

但是，怎麼沒有抓到IP address?

原來SuSe這個Linux OS就和我20年前剛接觸Linux一樣，每個網路介面都需要為它寫一個設定檔

```
$ cd /etc/sysconfig/network
$ touch ifcfg-eth0
$ vi ifcfg-eth0
IPADDR=''
NETMASK=''
NETWORK=''
STARTMODE='auto'
BOOTPROTO='dhcp'
USERCONTROL=no
FIREWALL=no
$ cp ifcfg-eth0 ifcfg-eth1
```

再重啟網路

`$ rcnetwork restart`

可以上網了