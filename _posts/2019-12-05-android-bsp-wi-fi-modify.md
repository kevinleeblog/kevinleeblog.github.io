---
layout: post
title: "Android Wi-Fi過濾(上)"
auther: Kevin Lee
category: 
tags: [Android_ROM_Design,Job_Logging]
subtitle: Android-7.1.2
visualworkflow: true
---

### 為何做？

最近接到一個任務要評估是否Android的Wi-Fi訊號，只列出感興趣的那個AP，其他都不出現在選項上
因為有些Android要做成某種產品時，希望能夠控管或限定接收某些公司提供的AP，其他都過濾不顯示。

### 如何做

想法就是先尋找Setting -> Wi-Fi的程式碼在何處，然後在那邊先把AP用Hard Code固定，其他都不濾掉。

這段程式位於*packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java*
最下面的AP hard code是"BQE_2.4G"
意思是說我只要這個ssid，其他都不顯示

```java
  public synchronized void onAccessPointsChanged() {
    
        final int wifiState = mWifiManager.getWifiState();

        switch (wifiState) {
            case WifiManager.WIFI_STATE_ENABLED:
                // AccessPoints are automatically sorted with TreeSet.
                final Collection<AccessPoint> accessPoints =
                        mWifiTracker.getAccessPoints();

                for (AccessPoint accessPoint : accessPoints) {
                	if(accessPoint.getSsidStr().equals("BQE_2.4G") == false)
                		continue;
```

編譯完後，趕緊來測試一下
果然只列出我感興趣的AP

![20191205_170058]({{site.baseurl}}/img/20191205_170058.jpg)

真實狀況的AP是這樣...

![image-20191205172017213]({{site.baseurl}}/img/image-20191205172017213.png)

### 後記

接下來要做的就是把過濾Wi-Fi AP做成一個System選項，可以讓使用者勾選是否啟用過濾，就留待下一集解謎