---
layout: post
title: "編譯Android User模式下安裝SerialPort App遇到的問題"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design, Job_Logging]
subtitle: "Using Google SerialPort App"
visualworkflow: true
---

### 為什麼要做

公司最近分配一個Task給我，要我去把Image編譯成User(之前都是Userdebug)的狀況下去驗證功能，編譯完並燒在公司板子(SC-60)上後發現SerialPort這隻app一執行馬上回報錯誤(在Userdebug是正常的)，所以就變成要去尋找錯誤原因。
SerialPort是Google提供的，且是整合在aosp內
https://code.google.com/archive/p/android-serialport-api/

![KCeuMqS]({{site.baseurl}}/img/KCeuMqS.png)

但由於Google提供的版本偏舊，是早期使用Eclipse開發，現在的Android IDE都是Android Studio
所以需要自行把code移轉至Android Studio
網路上已經有人移轉過去了，為了省時，我也直接使用現成的
但個人覺得時間允許的話了解怎麼轉移也是一種Know-How
https://github.com/licheedev/Android-SerialPort-API

### 要做什麼

目的就是Image編譯成User後，開啟SerialPort app去驗證Barcode scanner(/dev/ttyHSL1)與MCU Firmware Update(/dev/ttyHSL2)功能要正常

### 如何去做

這一系列全部解決後，大致上歸納會遇到三個問題

1. User模式下SerialPort app沒有出現ttyHSL1和ttyHSL2
2. User模式下SerialPort app沒有權限access ttyHSL1/ttyHSL2，即使ttyHSLx權限已經設成666
3. User模式下SerialPort app一開啟會跳出停止運作

因為是公司板子所以有預載一些特別driver，才會有ttyHSL1/ttyHSL2
如果是從Google下載的aosp本身是沒有的，但也無彷，我學習時也是拿aosp搭配emulator來練習

#### 問題1 User模式下SerialPort app沒有出現ttyHSL1和ttyHSL2

首先解決問題1沒有ttyHSL1/ttyHSL2的問題
經Trace後發現
sc-60在編譯成Userdebug和User使用的kernel config是不同的

```
$ vim device/qcom/msm8953_64/AndroidBoard.mk
```

![Txr7ozC]({{site.baseurl}}/img/Txr7ozC.png)

當編譯成user時，kernel config載入的是msmcortex-perf_defconfig
而編譯成eng或userdebug則是載入msmcortex_defconfig
發現是缺少了這兩個config，加到msmcortex-perf_defconfig就好

```
CONFIG_SERIAL_MSM_HSL=y
CONFIG_SERIAL_MSM_HSL_CONSOLE=y
```

#### 問題2 User模式下SerialPort app沒有權限access ttyHSL1/ttyHSL2，即使ttyHSLx權限已經設成666

User模式下SELinux預設是Enforcing，可以由logcat中發現SerialPort是觸發哪些permission被拒絕
以我的案例是SerialPort在User模式下無法access"/dev"目錄與無法access “/dev/ttyHSL1"和”/dev/ttyHSL2"這兩個Device node
解決方式是定義ttyHSL1和ttyHSL2的Type為quec_device

```
$ vim device/qcom/sepolicy/common/file_contexts
/dev/ttyHSL1                                    u:object_r:quec_device:s0
/dev/ttyHSL2                                    u:object_r:quec_device:s0
```

quec_device因為quectel已經預先定義成mlstrustedobject，所以我就不自創新的Type直接拿來用

```
$ cat device/qcom/sepolicy/common/device.te
type quec_device, dev_type, mlstrustedobject;
```

在domain.te內加上，在domain.te表示任何process都能access"/dev"目錄和quec_device

```
$ device/qcom/sepolicy/common/domain.te
allow domain device:dir {read write open};
allow domain quec_device:chr_file {open read ioctl write};
```

#### 問題3 User模式下SerialPort app一開啟會跳出停止運作

這個問題奇怪的地方就是包到aosp內會無法執行，但若是透過adb install則是可以正常執行
根據錯誤訊息應該是說SerialPort因為有使用到JNI的libserial_port.so，因為Android在預設路徑上無法access到這個library所以立即錯誤跳出
若是使用adb install
會在/data/app/SerialPort/lib/libserial_port.so發現此so
但若是包在aosp則無

解決方式就是把SerialPort使用Android Studio打開後關閉Instance Run
並重新編譯得到的apk就無此問題

> Android Studio需要下載3.0的版本才有Instance Run選項可以關閉

Android SDK勾選CMAKE, LLDB和NDK

![Str9Tiq]({{site.baseurl}}/img/Str9Tiq.png)

打開SerialPort project
取消Instance Run

![0QWsk4d]({{site.baseurl}}/img/0QWsk4d.png)

重新編譯