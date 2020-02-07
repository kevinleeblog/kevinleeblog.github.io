---
layout: post
title: "Porting USB Over Serial to Android 7"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design,Job_Logging]
subtitle:
visualworkflow: true
---

繼續承[Porting USB Over Ethernet to Android 7](/2019/12/25/porting-usb-over-ethernet-to-android-7/)
接下來要Porting的Chip是USB Over Serial，分別是使用Prolific PL2303G和ASIX MCS7840，這兩個Chip在AOSP預設都是沒有預載Driver，所以第一件事就是把Driver給重新編譯進來

*kernel/msm-3.18/arch/arm64/configs/msmcortex-perf_defconfig*
*kernel/msm-3.18/arch/arm64/configs/msmcortex_defconfig*

```
CONFIG_USB_SERIAL_MOS7840=y
CONFIG_USB_SERIAL_PL2303=y
```

這邊要注意的是PL2303G是2019年新出的Chip，目前的PL2303 Driver都是舊版的並沒有包含到PL2303G的USB PID，這邊我是先和代理商那邊要code然後打Patch上去

另外在Android Framework這邊也要設置，當USB列舉時會產生裝置檔ttyUSBx，如果沒有特別設置的話，權限預設都是root，會使一般使用者無法使用，所以需要額外設置權限把root換成systeme給ttyUSBx

*device/qcom/common/rootdir/etc/ueventd.qcom.rc*

```
 /dev/ttyUSB0              0660   system     system
 /dev/ttyUSB1              0660   system     system
 /dev/ttyUSB2              0660   system     system
```

以上做完，就可以在Android userdebug模式下使用USB Over Serial
要在Android user模式下使用還須設置Selinux權限

*device/qcom/sepolicy/common/file_contexts*

```
/dev/ttyUSB[0-9]*                              u:object_r:quec_device:s0
```

*device/qcom/sepolicy/common/domain.te*

```
allow domain device:dir {read write open};
allow domain quec_device:chr_file {open read ioctl write};
```

*device/qcom/sepolicy/common/device.te*

```
type quec_device, dev_type, mlstrustedobject;
```

