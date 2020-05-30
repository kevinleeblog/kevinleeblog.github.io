---
layout: post
title: "Solve QA Issue: non -staticdev package contains static .a library"
author:     Kevin Lee
tags: 		[OpenBMC] 
subtitle:   
category:  project1
visualworkflow: true
---
最近在寫Yocto recipes時，如果有包含到static library，那麼當在do_package_qa時，就會跳出此錯誤。

![image-20190114161624688](../img/image-20190114161624688-7453784.png)

搜尋網路上的解決方式就是在bb file內加入下行

```
INSANE_SKIP_${PN}-dev = "staticdev"
```

表示static library不做QA Check