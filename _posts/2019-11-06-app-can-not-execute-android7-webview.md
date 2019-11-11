---
layout: post
title: "分析App不能執行在android-7.1.2 webview"
auther: Kevin Lee
category: project1
tags: [Android, Job_Logging]
subtitle:
visualworkflow: true
---

### 為何做？

之前解決客戶的DeviceOwner問題後，再次詢問了一個問題
![image-20191106111754112]({{site.baseurl}}/img/image-20191106111754112.png)

大意是說，客戶的app在版本比較低的Webview上執行會有問題，換到較新版本的webview上則動作正常。
但是現在要使用我們自家生產的Android平板上跑客戶的app，目前確定是已經安裝測試了，有發生問題，而Webview的版本也是屬於版本比較低的，但是客戶想要了解是不是只有Webview造成，還有其他問題嗎，這個Job才會分到我這

### 如何做？

首先，自家生產Android7.1.2 Webview版本為52.0.2743.100
客戶有把logcat file給我，其中有如下一段

```
26512:02-03 04:31:51.276  5189  5189 I chromium: [INFO:CONSOLE(418)] "Uncaught SyntaxError: missing ) after argument list", source: file:///sdcard/Download/ZSOPIC/so_control.js (418)
26513:02-03 04:31:51.279  5189  5189 I chromium: [INFO:CONSOLE(76)] "Uncaught SyntaxError: Unexpected token function", source: file:///sdcard/Download/ZSOPIC/timer.js (76)
26514:02-03 04:31:51.279  5189  5189 I chromium: [INFO:CONSOLE(4)] "Uncaught SyntaxError: Unexpected token function", source: file:///sdcard/Download/ZSOPIC/header.js (4)
26515:02-03 04:31:51.281  5189  5189 I chromium: [INFO:CONSOLE(4)] "Uncaught SyntaxError: Unexpected token function", source: file:///sdcard/Download/ZSOPIC/body-menu.js (4)
26516:02-03 04:31:51.294  5189  5189 I chromium: [INFO:CONSOLE(208)] "Uncaught SyntaxError: missing ) after argument list", source: file:///sdcard/Download/ZSOPIC/customize-choose-number.js (208)
26517:02-03 04:31:51.301  5189  5189 I chromium: [INFO:CONSOLE(6)] "Uncaught SyntaxError: Unexpected token function", source: file:///sdcard/Download/ZSOPIC/body-orderlist.js (6)
26518:02-03 04:31:51.308  5189  5189 I chromium: [INFO:CONSOLE(84)] "Uncaught SyntaxError: Unexpected token function", source: file:///sdcard/Download/ZSOPIC/body-confirmation.js (84)
26519:02-03 04:31:51.308  5189  5189 I chromium: [INFO:CONSOLE(49)] "Uncaught SyntaxError: Unexpected token function", source: file:///sdcard/Download/ZSOPIC/footer.js (49)
26520:02-03 04:31:51.311  5189  5189 I chromium: [INFO:CONSOLE(3)] "Uncaught SyntaxError: missing ) after argument list", source: file:///sdcard/Download/ZSOPIC/ei-self.js (3)
26521:02-03 04:31:51.313  5189  5189 I chromium: [INFO:CONSOLE(5)] "Uncaught SyntaxError: Unexpected token (", source: file:///sdcard/Download/ZSOPIC/top.js (5)
```

Javascript在Webview version 52上執行會有問題
客戶的Javascriptd可能有使用到Async function，而目前Webview version 55以上才支援這個功能

[https://caniuse.com/#search=async%20function](https://caniuse.com/#search=async function)

![image-20191106133226568]({{site.baseurl}}/img/image-20191106133226568.png)

所以應該99.9%確認是Webview版本問題