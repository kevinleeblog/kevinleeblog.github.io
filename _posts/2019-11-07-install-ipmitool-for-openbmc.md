---
layout: post
title: "OpenBMC Install ipmitool"
auther: Kevin Lee
category: 
tags: [OpenBMC]
subtitle:
visualworkflow: true
---

### 為什麼要做？

OpenBMC編譯並使用Qemu啟動後，接下來就是開始使用ipmitool去了解ipmi觀念與程式設計。
但目前是使用Qemu，還不知道如何把Qemu的網路弄通讓它可以和Host主機溝通，所以先把ipmitool安裝在Qemu內的OpenBMC上，然後走IPMB和OpenBMC溝通。

### 如何做？

安裝ipmitool package給OpenBMC上

```
$ cd openbmc
$ . openbmc-env
//Add this new line
$ vim conf/local.conf
---
IMAGE_INSTALL_append = " ipmitool "
---
$ bitbake obmc-phosphor-image
```

重新編譯好後於*tmp/deploy/image/romulus* 執行Qemu

```
$ qemu-system-arm -m 256 -M romulus-bmc -nographic -drive file=./obmc-phosphor-image-romulus.static.mtd,format=raw,if=mtd -net nic -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostname=qemu
```

登入後，傳送IPMI command: 06 01

```
root@romulus:~# ipmitool raw 06 01
 00 80 00 00 02 8d 00 00 00 00 00 00 00 00 00
```

