---
layout: post
title: "Android NDK - standalone toolchain"
auther: Kevin Lee
category: project1
tags: [Android]
subtitle:
visualworkflow: true
---

這個任務的源由是韌體組想把NXP提供的MCU燒錄程式放到Android BSP上編譯並成為Android console內可執行的執行檔，這個燒錄程式的Source為使用Makefile的GNU編譯

使用Android NDK編譯C或C++有兩種方式，其一是根據Makefile創造Android.mk並透過aosp編譯產生執行檔，另一種就是下載NDK包並產生Standalone toolchain搭配Makefile來產生執行檔

本文介紹使用Standalone Toolchain

## 安裝Standalone Toolchain

https://developer.android.com/ndk/downloads/index.html?hl=zh-tw

![image-20200507092722677]({{site.baseurl}}/img/image-20200507092722677.png)

```
$ unzip android-ndk-r21b-linux-x86_64.zip
$ cd android-ndk-r21b
$ build/tools/make_standalone_toolchain.py --arch arm --api 28 --install-dir /tmp/my-android-toolchain
```

因為我的BSP為Android 9 Base，所以我的API Level為28

接下來就是設定一些環境變數

```
# Add the standalone toolchain to the search path.
export PATH=$PATH:/tmp/my-android-toolchain/bin

# Tell configure what tools to use.
target_host=aarch64-linux-android
export AR=$target_host-ar
export AS=$target_host-clang
export CC=${target_host}28-clang
export CXX=${target_host}28-clang++
export LD=$target_host-ld
export STRIP=$target_host-strip

# Tell configure what flags Android requires.
export CFLAGS="-fPIE -fPIC"
export LDFLAGS="-pie
```

可以開始編譯程式了