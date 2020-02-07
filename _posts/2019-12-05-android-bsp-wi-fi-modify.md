---
layout: post
title: "做一個在Android只列出想要的Wi-Fi AP(一)"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design,Job_Logging]
subtitle: Android-7.1.2
visualworkflow: true
---

### 為何做？

最近接到一個任務要評估是否Android的Wi-Fi訊號，只列出想要的那個AP，其他都不出現在WiFi選項上。
應用場合是因為Android要做成某種產品時，例如點餐機時，希望那個平板能夠控管或限定接收某些公司提供的WiFi AP，其他都過濾不顯示。

### 如何做

想法就是先尋找Setting -> Wi-Fi的頁面程式碼位於AOSP何處，然後在那邊先把某一AP用Hard Code固定
這段程式位於*packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java*
最下面的AP hard code是"BQE_2.4G"
意思是說我只要這個ssid，其他都不顯示。

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

接下來要做的就是把過濾Wi-Fi AP做成一個開關放在Wifi Setting，可以讓使用者勾選是否啟用固定AP，以及把AP Hard Code變成一種可以給客戶選擇想要固定的AP
就留待下一集解謎...