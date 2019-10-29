---
layout: post
title: "Android System App for AOSP"
auther: Kevin Lee
category: 
tags: [Android-ROM-Custom]
subtitle:
visualworkflow: true
---

### 為何要做？

Android system有一些權限的控管，成為System app後能夠避免一些權限不足而無法使用的問題



### 如何做？

打開Android Studio開啟專案的AndroidManifest.xml
加上android:sharedUserId=“android.uid.system”
![lRYXIgg]({{site.baseurl}}/img/lRYXIgg.png)

BSP安裝app的Android.mk

```Android.mk
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
# Module name should match apk name to be installed.
LOCAL_MODULE := SerialPort_1.1
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := platform
LOCAL_DEX_PREOPT := false
include $(BUILD_PREBUILT)
```

LOCAL_CERTIFICATE := platform
加上LOCAL_CERTIFICATE := platform
不加LOCAL_DEX_PREOPT := false會產生dex2oatd編譯錯誤
![E4IVP1P]({{site.baseurl}}/img/E4IVP1P.png)

網路上查是說APK是使用較高版本的SDK Version(API Level 28)編譯生成，然後放到BSP(API Level 25)編譯產生錯誤
程式編譯會對apk進行解析，形成odex文件加速apk的運行，但是高版本的SDK開發的Apk一些資源無法被低版本正確的解析