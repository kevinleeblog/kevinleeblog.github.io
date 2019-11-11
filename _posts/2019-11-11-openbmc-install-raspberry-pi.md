---
layout: post
title: "OpenBMC Installed On Raspberry Pi"
auther: Kevin Lee
category: 
tags: [OpenBMC]
subtitle:
visualworkflow: true
---

#### 為何要做？

因為想要使用IPMI透過RMCP與OpenBMC溝通，但這部分在Qemu上的網路不會設定，所以想把OpenBMC放在Raspberry Pi上執行

#### 對Raspberry Pi如何設定

```
$ TEMPLATECONF=meta-evb/meta-evb-raspberrypi/conf . openbmc-env
$ bitbake obmc-phosphor-image
```

#### Issue: Image is too larger

剩下最後一個步驟就是要做Image時出現error

```
$ bitbake obmc-phosphor-image
Loading cache: 100% |##################################################################################################################################| Time: 0:00:00
Loaded 3556 entries from dependency cache.
Parsing recipes: 100% |################################################################################################################################| Time: 0:00:01
Parsing of 2423 .bb files complete (2422 cached, 1 parsed). 3557 targets, 341 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies

Build Configuration:
BB_VERSION           = "1.44.0"
BUILD_SYS            = "x86_64-linux"
NATIVELSBSTRING      = "ubuntu-16.04"
TARGET_SYS           = "arm-openbmc-linux-gnueabi"
MACHINE              = "raspberrypi"
DISTRO               = "openbmc-phosphor"
DISTRO_VERSION       = "0.1.0"
TUNE_FEATURES        = "arm armv6 vfp arm1176jzfs callconvention-hard"
TARGET_FPU           = "hard"
meta                 
meta-poky            
meta-oe              
meta-networking      
meta-python          
meta-webserver       
meta-phosphor        
meta-raspberrypi     = "master:f3f93bb878a10643895ece0c3926547b853dff7b"

Initialising tasks: 100% |#############################################################################################################################| Time: 0:00:01
Sstate summary: Wanted 3 Found 1 Missed 2 Current 1530 (33% match, 99% complete)
NOTE: Executing Tasks
NOTE: Setscene tasks completed
ERROR: obmc-phosphor-image-1.0-r0 do_generate_static: Image '/home/kevinlee/openbmc/build/tmp/deploy/images/raspberrypi/fitImage-obmc-phosphor-initramfs-raspberrypi-raspberrypi' is too large!
ERROR: Logfile of failure stored in: /home/kevinlee/openbmc/build/tmp/work/raspberrypi-openbmc-linux-gnueabi/obmc-phosphor-image/1.0-r0/temp/log.do_generate_static.23235
ERROR: Task (/home/kevinlee/openbmc/meta-phosphor/recipes-phosphor/images/obmc-phosphor-image.bb:do_generate_static) failed with exit code '1'
NOTE: Tasks Summary: Attempted 4393 tasks of which 4392 didn't need to be rerun and 1 failed.

Summary: 1 task failed:
  /home/kevinlee/openbmc/meta-phosphor/recipes-phosphor/images/obmc-phosphor-image.bb:do_generate_static
```

上網看看OpenBMC的人員的建議說是因為OpenBMC有太多Packages，所以塞不進Raspberry Pi
建議移除一些套件
https://github.com/openbmc/openbmc/issues/3590

![image-20191111135322941]({{site.baseurl}}/img/image-20191111135322941.png)

其實我有照建議把bmcweb給移除，但是還是"*Image is too large*" error。

但我個人是覺得納悶，因為Raspberry Pi是使用SD Card當作儲存空間，所以不可能塞不進去，應是Yocto有地方設定限制了Image容量
最後我改了*image_types_phosphor.bbclass*，把Image 32MB換成128MB就可以Pass

```diff
$ git diff
diff --git a/meta-phosphor/classes/image_types_phosphor.bbclass b/meta-phosphor/classes/image_types_phosphor.bbclass
index f7742c8..28035d1 100644
--- a/meta-phosphor/classes/image_types_phosphor.bbclass
+++ b/meta-phosphor/classes/image_types_phosphor.bbclass
@@ -28,7 +28,7 @@ IMAGE_TYPES_MASKED += "mtd-static mtd-static-alltar mtd-static-tar mtd-ubi mtd-u
 
 # Flash characteristics in KB unless otherwise noted
 DISTROOVERRIDES .= ":flash-${FLASH_SIZE}"
-FLASH_SIZE ?= "32768"
+FLASH_SIZE ?= "131072"
 FLASH_PEB_SIZE ?= "64"
 # Flash page and overhead sizes in bytes
 FLASH_PAGE_SIZE ?= "1"
```

接下來就是要燒到SD Card上看可不可以開機了。

