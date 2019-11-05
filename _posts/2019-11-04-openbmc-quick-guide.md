---
layout: post
title: "OpenBMC Basic-Compile and Quem(一)"
auther: Kevin Lee
category: project1
tags: [OpenBMC]
subtitle:
visualworkflow: true
---

### 為何做？

在AMI工作時，我把BMC看待就如同Embedded Linux上一樣，說來慚愧，我對Linux很了解，但我不是很完全了解BMC這個行業，AMI的BMC發展已經很成熟，市佔率已經90％以上，一般想要跨進Server這行業，大多都是使用AMI的Solution，另外有部分的公司也對另外一種免費的Solution很有興趣就是OpenBMC，但往往只知其名，不知其意義，因為整個BMC圍繞的核心就是IPMI，Intel把IPMI應用在BMC上很透徹，所以想要進入BMC，就需要對IPMI熟悉，就像是想要開發藍牙，也要對藍牙規範有一定程度了解，但難就在於IPMI文件內有許多專有名詞，必須要對專有名詞的意涵有些認識。

雖然我已離開BMC這行業了，但生活中很多的遠端連線的範例其實都和BMC的意義是相通，所以才想去發自內心去了解它

### 如何編譯？

整個OpenBMC都是Open和Free的
https://github.com/openbmc/openbmc
編譯之前，有些套件要先安裝

> 現在回頭看OpenBMC，發現有些步驟和之前學習時有異，所以現在的安裝步驟也許將來也會有些微的修正

### 編譯環境

##### Ubuntu-16.04

**Packages**

`$ sudo apt-get install -y git build-essential libsdl1.2-dev texinfo gawk chrpath diffstat`

**Node-js**
主要是在編譯WebUI時會用到

```
$ sudo apt-get install nodejs
$ sudo npm cache clean --force
```

**Download OpenBMC Source**

```
$ git clone git@github.com:openbmc/openbmc.git
$ cd openbmc
```

以我當時下載的commit log: f3f93bb878a10643895ece0c3926547b853dff7b

> 選擇Hardware Platform，根據官網示範範例是使用Romulus
> `$ TEMPLATECONF=meta-ibm/meta-romulus/conf . openbmc-env`
> 其餘Hardware Platform可藉由`find meta-* -name local.conf.sample`搜尋

#### 開始編譯

`$ bitbake obmc-phosphor-image`

第一下編譯會需要很久的時間
這段時間可以先下載Qemu，等下編譯完後，就可以拿到image去執行模擬。

## Download Qemu

```
$ wget https://openpower.xyz/job/openbmc-qemu-build-merge-x86/lastSuccessfulBuild/artifact/qemu/arm-softmmu/qemu-system-arm
$ chmod u+x qemu-system-arm
$ sudo mv qemu-system-arm /usr/local/bin
```

### Start Qemu

剛剛編譯好的image會放在*openbmc/build/tmp/deploy/images/romulus*
到此目錄下執行Qemu Command

```
$ qemu-system-arm -m 256 -M romulus-bmc -nographic -drive file=./obmc-phosphor-image-romulus.static.mtd,format=raw,if=mtd -net nic -net user,hostfwd=:127.0.0.1:2222-:22,hostfwd=:127.0.0.1:2443-:443,hostname=qemu



U-Boot 2016.07 (Nov 04 2019 - 09:50:48 +0000)

       Watchdog enabled
DRAM:  240 MiB
Flash: 32 MiB
*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
Net:   aspeednic#0
Error: aspeednic#0 address not set.

Hit any key to stop autoboot:  0 
## Loading kernel from FIT Image at 20080000 ...
   Using 'conf@aspeed-bmc-opp-romulus.dtb' configuration
   Trying 'kernel@1' kernel subimage
     Description:  Linux kernel
     Type:         Kernel Image
     Compression:  uncompressed
     Data Start:   0x20080128
     Data Size:    2731552 Bytes = 2.6 MiB
     Architecture: ARM
     OS:           Linux
     Load Address: 0x80001000
     Entry Point:  0x80001000
     Hash algo:    sha256
     Hash value:   356a9dbb12554b6f9c83509bb992e16bd6ce4fdc39fe0b477da11e2a1d5d31d3
   Verifying Hash Integrity ... sha256+ OK
## Loading ramdisk from FIT Image at 20080000 ...
   Using 'conf@aspeed-bmc-opp-romulus.dtb' configuration
   Trying 'ramdisk@1' ramdisk subimage
     Description:  obmc-phosphor-initramfs
     Type:         RAMDisk Image
     Compression:  uncompressed
     Data Start:   0x20323770
     Data Size:    1116140 Bytes = 1.1 MiB
     Architecture: ARM
     OS:           Linux
     Load Address: unavailable
     Entry Point:  unavailable
     Hash algo:    sha256
     Hash value:   f134181205bcfafd9a8b4fe67a88705ec7ae001fe2ec7403d27d9f52c3330015
```

登入預設帳號/密碼：root/0penBmc

## 初步使用OpenBMC( With Qemu)

#### SSH登入

```
$ ssh -p 2222 root@127.0.0.1
```

> 會使用到Port 2222是，官網是說因為他們有使用到Jenkins和一些自動測試程式會使用到SSH和Https的Port22和Port423
>
> 所以使用Qemu就需要Port Forward到2222和2443個別對應到SSH和HTTPS



#### WebUI登入

網址：https://127.0.0.1:2443

![image-20191105102740005]({{site.baseurl}}/img/image-20191105102740005.png)

![image-20191105102956161]({{site.baseurl}}/img/image-20191105102956161.png)

##### Check Machine State

```
root@romulus:~# obmcutil state
CurrentBMCState     : xyz.openbmc_project.State.BMC.BMCState.Ready
CurrentPowerState   : xyz.openbmc_project.State.Chassis.PowerState.Off
CurrentHostState    : xyz.openbmc_project.State.Host.HostState.Off
BootProgress        : xyz.openbmc_project.State.Boot.Progress.ProgressStages.Unspecified
OperatingSystemState: xyz.openbmc_project.State.OperatingSystem.Status.OSStatus.Inactive
```

離開Qemu:`ctrl+a x`

