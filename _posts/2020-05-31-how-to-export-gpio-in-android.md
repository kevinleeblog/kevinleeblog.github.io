---
layout: post
title: "How to export GPIO in Android Open Source Project"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle: 
visualworkflow: true
---

有時候會需要用到SOM上的GPIO用來控制某些裝置或是監聽，預設的Android作業系統不會特意打開SOM上的GPIO，須自行手動加入到AOSP內。

另外，GPIO也不是想開那根就隨意開那根，還是需要根據Pin defined上那些可以被設成GPIO的才能被使用

![image-20200531010213960]({{site.baseurl}}/img/image-20200531010213960.png)

範例1: GPIO_89設成Output

*device/qcom/msm8953_64/init.target.rc*

```
on boot

    write /sys/class/gpio/export 89
    write /sys/class/gpio/gpio89/direction out 
    write /sys/class/gpio/gpio89/value 1
    chown system system /sys/class/gpio/gpio89/value
    chmod 0666 /sys/class/gpio/gpio89/value
```

範例2: GPIO_36設成Input

*device/qcom/msm8953_64/init.target.rc*

    on boot
    
    write /sys/class/gpio/export 36
    write /sys/class/gpio/gpio36/direction in
    chown system system /sys/class/gpio/gpio36/value
    chmod 0666 /sys/class/gpio/gpio36/value