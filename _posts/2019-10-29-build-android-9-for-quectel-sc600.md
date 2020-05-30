---
layout: post
title: "Build Android 9 for Quectel SC600"
auther: Kevin Lee
category: project1
tags: [ Android Open Source Project, MSM8953, Qualcomm]
subtitle: 
visualworkflow: true
---

介紹如何在Ubuntu16.04的環境下編譯

#### Install Packages

1. Install software packages

```
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev libc6-dev lib32ncurses5-dev x11proto-core-dev libx11-dev libgl1-mesa-dev g++-multilib tofrodos python-markdown libxml2-utils xsltproc
```

2. Install mingw32 and lib32z-dev

```
$ sudo vim /etc/apt/sources.list
//在sources.list末尾添加
$ sudo apt-get update
$ sudo apt-get install mingw32 lib32z-dev
```

3. Install Java8

```
$ sudo apt-get install openjdk-8-jdk
```

#### Download Google Android 9.0.0 kernel 4.9

```
$ repo init -u https://source.codeaurora.org/quic/la/platform/manifest.git -b release -m LA.UM.7.6.2.r1-04300-89xx.0.xml --repo-url=git://codeaurora.org/tools/repo.git --repo-branch=caf-stable
$ repo sync
```

Download SC600 platform dependent SDK ‘*SC600_Android9.0.0_Quectel_SDK_r000120_20190610.tar.gz*’, decompress and overwrite the Google Android 9.0.0 kernel 4.9

#### Start to Build

```
$ source build/envsetup.sh
$ lunch msm8953_64-userdebug
$ make -j 6
```

編譯過程中若有報錯缺少opensslv.h

```
CHK     include/config/kernel.release
  GEN     ./Makefile
  CHK     include/generated/uapi/linux/version.h
  CHK     include/generated/utsrelease.h
  Using /home/kevinlee/protech_workspace/android_source/MH5108_SC600/kernel/msm-4.9 as source for kernel
  CHK     scripts/mod/devicetable-offsets.h
  HOSTCC  scripts/sign-file
/home/kevinlee/protech_workspace/android_source/MH5108_SC600/kernel/msm-4.9/scripts/sign-file.c:25:30: fatal error: openssl/opensslv.h: 沒有此一檔案或目錄
compilation terminated.
scripts/Makefile.host:101: recipe for target 'scripts/sign-file' failed
make[2]: *** [scripts/sign-file] Error 1
/home/kevinlee/protech_workspace/android_source/MH5108_SC600/kernel/msm-4.9/Makefile:562: recipe for target 'scripts' failed
```

安裝缺少套件

```
$ sudo apt-get install libssl-dev
```