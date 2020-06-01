---
layout: post
title: "Qualcomm Ramdump使用"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle: 
visualworkflow: true
---

使用Qualcomm系列開發一直有個很麻煩的問題，就是當系統不知莫名原因發生造成系統死當以至於啟動Watchdog重開機，這中間你完全看不到任何panic訊息，造成Debug的難度。所以Ramdump就是當系統死當時，下載死當期間的ram到PC端，然後再去分析並得到panic是死在那。

### 安裝Ramdump工具

1. 安裝QPST工具集
2. 安裝Qualcomm USB HS-USB Diagnostics 9091驅動程式

### 觸發Ramdump

手機線接電腦，使用Ramdump的手機要有root，不然會權限不夠無法執行下述命令

```
$ adb shell
msm8953_64:/ $ su
msm8953_64:/ # cat /sys/module/msm_poweroff/parameters/download_mode           
0
msm8953_64:/ # echo 1 > /sys/module/msm_poweroff/parameters/download_mode   
```

故意觸發死當

```
msm8953_64:/ # echo c > /proc/sysrq-trigger
```

執行完畢後，啟動*QPST Configuration*

切到Port頁面可以看到進度條，表示目前這在從RAM中抓取資料

![image-20200601105554803]({{site.baseurl}}/img/image-20200601105554803.png)

跑完後的狀態會變成這樣，無任何提示

![image-20200601105819660]({{site.baseurl}}/img/image-20200601105819660.png)

其實Ramdump已經抓下來啦，要這樣開

![image-20200601110010051]({{site.baseurl}}/img/image-20200601110010051.png)

![image-20200601110122174]({{site.baseurl}}/img/image-20200601110122174.png)

### 分析Ramdump

以上的工具都是代理商提供的，所以應該不至於會有太大的問題，但接下來要如何使用分析工具尚不知道