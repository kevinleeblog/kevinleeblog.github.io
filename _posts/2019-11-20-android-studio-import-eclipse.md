---
layout: post
title: "Android Studio匯入Eclipse ADT專案"
auther: Kevin Lee
category: project1
tags: [Android]
subtitle:
visualworkflow: true
---

### 為什麼做？

今天assign的工作，說要測試廠商給的Android Demo apk以及Source Code，去官網下載Source Code後，發現是使用eclipse IDE開發的，但是現在開發Android幾乎都是使用Android Studio，所以就必須要把Source Code的專案轉換到Android Studio

### 如何做？

打開Android Studio，然後選擇`Import project(Gradle, Eclipse ADT, etc.)`

![image-20191120173055427]({{site.baseurl}}/img/image-20191120173055427.png)

選擇要轉換的專案目錄位置並按Next到Finish

![image-20191120173305434]({{site.baseurl}}/img/image-20191120173305434.png)

在我的case裡，出現第一個error
`ERROR: Could not find com.android.tools.build:gradle:3.5.0.`

點選紅圈圈建議的處理方式

![image-20191120173715476]({{site.baseurl}}/img/image-20191120173715476.png)

![image-20191120173841225]({{site.baseurl}}/img/image-20191120173841225.png)

繼續Run!!!!!

出現第二個錯誤 `ERROR: Minimum supported Gradle version is 5.4.1. Current version is 4.8.`
一樣選擇紅圈建議，繼續Run

![image-20191120174206787]({{site.baseurl}}/img/image-20191120174206787.png)

出現第三個錯誤`ERROR: The minSdk version should not be declared in the android manifest file. You can move the version from the manifest to the defaultConfig in the build.gradle file.`
一樣建議方式處理

![image-20191120174500954]({{site.baseurl}}/img/image-20191120174500954.png)

最後總算轉換過來了