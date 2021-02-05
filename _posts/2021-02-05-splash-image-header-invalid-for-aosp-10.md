---
layout: post
title: "Splash image header invalid for AOSP 10"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle:
visualworkflow: true
---

最近在把自家產品從Android 9升到10遇到一些問題，其中一個就是開機Logo無法載入，一直都是預設的小企鵝，但如果使用Qfil燒回Android 9又可正常顯示，看開機log時有出現一行「ERROR: Splash image header invalid」

一開始以為是splash.img燒到splash partition可能在開機階段被改掉導致header invalid錯誤發生，所以先檢查是不是真的被修改了

### 下載機器的splash.img檔

接好adb，打開Qfil並開機

![image-20210205135212652]({{site.baseurl}}/img/image-20210205135212652.png)

![image-20210205135700332]({{site.baseurl}}/img/image-20210205135700332.png)

![image-20210205135732503]({{site.baseurl}}/img/image-20210205135732503.png)

![image-20210205135804987]({{site.baseurl}}/img/image-20210205135804987.png)

![image-20210205140043256]({{site.baseurl}}/img/image-20210205140043256.png)

和原先的splash.img來比較內容看有何差異

```
$ vimdiff ReadData_eMMC_Lun0_0xb0000_Len22528_DT_05_02_2021_13_58_31.bin splash.img
```

![image-20210205140328555]({{site.baseurl}}/img/image-20210205140328555.png)

換成binary

```
:%! xxd
```

![image-20210205140519949]({{site.baseurl}}/img/image-20210205140519949.png)

發現燒進去的和取出來的內容是一模一樣!

詢問代理商，給了我一包Qfil update包 