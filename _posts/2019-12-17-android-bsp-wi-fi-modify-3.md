---
layout: post
title: "做一個在Android只列出想要的Wi-Fi AP(三) - Using external storage"
auther: Kevin Lee
category: 
tags: [Android_ROM_Design,Job_Logging]
subtitle:
visualworkflow: true
---

上篇[做一個在Android只列出想要的Wi-Fi AP(二) - 使用SharedPreference](/2019/12/16/android-bsp-wi-fi-modify-2)失敗
主要就是Google認為使用此方法共享Resource會讓系統有個漏洞，所以在Android 5或之後的版本都已經改掉了。

現在還有兩個方法可以試試，其中一個是Google建議的使用ContentProvider，另外一個就是使用External Storage，External Storage我這邊已經已經突破了，所以就紀錄此方法如何達成。

### 首先從APP端說明

UI外觀沿用先前已經設計好的

![image-20191216102706198]({{site.baseurl}}/img/image-20191216102706198.png)

因為是使用到External Storage，所以app要增加權限

```
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

```java
package com.protech.os.wififilter;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.os.Environment;
import android.util.Log;
import android.util.Xml;
import android.view.View;
import android.widget.CompoundButton;
import android.widget.EditText;
import android.widget.Switch;

import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserException;
import org.xmlpull.v1.XmlSerializer;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.FileWriter;
import java.io.IOException;

public class MainActivity extends AppCompatActivity {

    final String SETTING_FILE_NAME = "wifi_filter_setting.xml";

    EditText ssidStringEditText;
    Switch wifiFilterEnableSwitch;

    String wifiEnableSwith;
    String ssid;

    File file;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        wifiFilterEnableSwitch = (Switch)findViewById(R.id.wifiFilterEnableSwitch);
        ssidStringEditText=(EditText)findViewById(R.id.ssidStringEditText);

        File sdCard = Environment.getExternalStorageDirectory();
        File dir = new File (sdCard.getAbsolutePath() + "/wifi_filter/");
        file = new File(dir, SETTING_FILE_NAME);

        if(dir.exists() == false) {
            dir.mkdir();

            try {
                wifiEnableSwith="0";
                ssid="";

                FileOutputStream fos = new  FileOutputStream(file);
                XmlSerializer xmlSerializer = Xml.newSerializer();
                xmlSerializer.setOutput(fos, "UTF-8");
                xmlSerializer.startDocument("UTF-8", true);
                xmlSerializer.startTag(null, "WIFI_FILTER_SWITCH");
                xmlSerializer.text(wifiEnableSwith);
                xmlSerializer.endTag(null, "WIFI_FILTER_SWITCH");
                xmlSerializer.startTag(null,"WIFI_FILTER_SSID");
                xmlSerializer.text(ssid);
                xmlSerializer.endTag(null, "WIFI_FILTER_SSID");
                xmlSerializer.endDocument();
                xmlSerializer.flush();
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }catch (IOException e) {
                e.printStackTrace();
            }

        } else {
            try {
                FileInputStream fis = new FileInputStream(file);
                XmlPullParser pullParser = Xml.newPullParser();
                pullParser.setInput(fis, "UTF-8");

                // Read the file flag
                int fileFlag = pullParser.getEventType();

                while(fileFlag != XmlPullParser.END_DOCUMENT){
                    String nodeName = pullParser.getName();
                    switch (fileFlag) {
                        case XmlPullParser.START_TAG:
                            if(nodeName.equals("WIFI_FILTER_SWITCH")){
                                wifiEnableSwith = pullParser.nextText();
                                Log.i("WifiFilter", "WIFI_FILTER_SWITCH = " + wifiEnableSwith);

                            }
                            if(nodeName.equals("WIFI_FILTER_SSID")){
                                ssid = pullParser.nextText();
                                Log.i("WifiFilter", "WIFI_FILTER_SSID = " + ssid);
                            }

                            break;
                        case XmlPullParser.END_TAG:
                        default:
                            break;
                    }

                    // Get the next event type
                    fileFlag = pullParser.next();
                }


            } catch (XmlPullParserException | IOException e) {
                e.printStackTrace();
            }
        }

        if(wifiEnableSwith.equals("1") == true) {
            wifiFilterEnableSwitch.setChecked(true);
            ssidStringEditText.setFocusable(false);
            ssidStringEditText.setFocusableInTouchMode(false);
        }
        else {
            wifiFilterEnableSwitch.setChecked(false);
        }
        ssidStringEditText.setText(ssid);

        wifiFilterEnableSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {

                FileOutputStream fos = null;

                if(isChecked == true)
                {
                    ssidStringEditText.setFocusable(false);
                    ssidStringEditText.setFocusableInTouchMode(false);

                    try {
                        fos = new FileOutputStream(file);
                        XmlSerializer xmlSerializer = Xml.newSerializer();
                        xmlSerializer.setOutput(fos, "UTF-8");

                        xmlSerializer.startDocument("UTF-8", true);
                        xmlSerializer.startTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.text("1");
                        xmlSerializer.endTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.startTag(null,"WIFI_FILTER_SSID");
                        xmlSerializer.text(ssidStringEditText.getText().toString());
                        xmlSerializer.endTag(null, "WIFI_FILTER_SSID");
                        xmlSerializer.endDocument();
                        xmlSerializer.flush();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                else
                {
                    ssidStringEditText.setFocusable(true);
                    ssidStringEditText.setFocusableInTouchMode(true);
                    ssidStringEditText.requestFocus();

                    try {
                        fos = new FileOutputStream(file);
                        XmlSerializer xmlSerializer = Xml.newSerializer();
                        xmlSerializer.setOutput(fos, "UTF-8");

                        xmlSerializer.startDocument("UTF-8", true);
                        xmlSerializer.startTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.text("0");
                        xmlSerializer.endTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.startTag(null,"WIFI_FILTER_SSID");
                        xmlSerializer.text(ssidStringEditText.getText().toString());
                        xmlSerializer.endTag(null, "WIFI_FILTER_SSID");
                        xmlSerializer.endDocument();
                        xmlSerializer.flush();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
    }
}

```

APP端要負責的任務就是紀錄兩個

- 是否啟用WIFI AP過濾功能
- 欲顯示的WIFI AP字串

如何記錄？使用者操作UI時，每一個動作都會轉成XML格式的檔案*wifi_filter_setting.xml*並存在*/storage/emulator/0/wifi_filter*

另外就是初始化的任務，每一次執行app都會判斷*/storage/emulator/0/wifi_filter*是否存在，用來區別是否是使用者第一次使用嗎？如果是第一次使用，就會建置*wifi_filter_setting.xml*檔案和*wifi_filter*目錄，並先填好XML的格式，並預設變數
 wifiEnableSwith="0";
 ssid="";

```java
        File sdCard = Environment.getExternalStorageDirectory();
        File dir = new File (sdCard.getAbsolutePath() + "/wifi_filter/");
        file = new File(dir, SETTING_FILE_NAME);

        if(dir.exists() == false) {
            dir.mkdir();

            try {
                wifiEnableSwith="0";
                ssid="";

                FileOutputStream fos = new  FileOutputStream(file);
                XmlSerializer xmlSerializer = Xml.newSerializer();
                xmlSerializer.setOutput(fos, "UTF-8");
                xmlSerializer.startDocument("UTF-8", true);
                xmlSerializer.startTag(null, "WIFI_FILTER_SWITCH");
                xmlSerializer.text(wifiEnableSwith);
                xmlSerializer.endTag(null, "WIFI_FILTER_SWITCH");
                xmlSerializer.startTag(null,"WIFI_FILTER_SSID");
                xmlSerializer.text(ssid);
                xmlSerializer.endTag(null, "WIFI_FILTER_SSID");
                xmlSerializer.endDocument();
                xmlSerializer.flush();
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }catch (IOException e) {
                e.printStackTrace();
            }
```

如果先前*wifi_filter_setting.xml*檔案和*wifi_filter*目錄已經存在，則會Parsing XML的檔案，並把內容指定給變數wifiEnableSwith和ssid

```java
        } else {
            try {
                FileInputStream fis = new FileInputStream(file);
                XmlPullParser pullParser = Xml.newPullParser();
                pullParser.setInput(fis, "UTF-8");

                // Read the file flag
                int fileFlag = pullParser.getEventType();

                while(fileFlag != XmlPullParser.END_DOCUMENT){
                    String nodeName = pullParser.getName();
                    switch (fileFlag) {
                        case XmlPullParser.START_TAG:
                            if(nodeName.equals("WIFI_FILTER_SWITCH")){
                                wifiEnableSwith = pullParser.nextText();
                                Log.i("WifiFilter", "WIFI_FILTER_SWITCH = " + wifiEnableSwith);
                            }
                            if(nodeName.equals("WIFI_FILTER_SSID")){
                                ssid = pullParser.nextText();
                                Log.i("WifiFilter", "WIFI_FILTER_SSID = " + ssid);
                            }
                            break;
                        case XmlPullParser.END_TAG:
                        default:
                            break;
                    }
                    // Get the next event type
                    fileFlag = pullParser.next();
                }
            } catch (XmlPullParserException | IOException e) {
                e.printStackTrace();
            }
        }
```

獲得變數wifiEnableSwith和ssid，就可以依照變數內容更換UI

```java
        if(wifiEnableSwith.equals("1") == true) {
            wifiFilterEnableSwitch.setChecked(true);
            ssidStringEditText.setFocusable(false);
            ssidStringEditText.setFocusableInTouchMode(false);
        }
        else {
            wifiFilterEnableSwitch.setChecked(false);
        }
        ssidStringEditText.setText(ssid);
```



當使用者操作UI時，也要把操作後的結果寫回到檔案內

```java
        wifiFilterEnableSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {

                FileOutputStream fos = null;

                if(isChecked == true)
                {
                    ssidStringEditText.setFocusable(false);
                    ssidStringEditText.setFocusableInTouchMode(false);

                    try {
                        fos = new FileOutputStream(file);
                        XmlSerializer xmlSerializer = Xml.newSerializer();
                        xmlSerializer.setOutput(fos, "UTF-8");

                        xmlSerializer.startDocument("UTF-8", true);
                        xmlSerializer.startTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.text("1");
                        xmlSerializer.endTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.startTag(null,"WIFI_FILTER_SSID");
                        xmlSerializer.text(ssidStringEditText.getText().toString());
                        xmlSerializer.endTag(null, "WIFI_FILTER_SSID");
                        xmlSerializer.endDocument();
                        xmlSerializer.flush();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                else
                {
                    ssidStringEditText.setFocusable(true);
                    ssidStringEditText.setFocusableInTouchMode(true);
                    ssidStringEditText.requestFocus();

                    try {
                        fos = new FileOutputStream(file);
                        XmlSerializer xmlSerializer = Xml.newSerializer();
                        xmlSerializer.setOutput(fos, "UTF-8");

                        xmlSerializer.startDocument("UTF-8", true);
                        xmlSerializer.startTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.text("0");
                        xmlSerializer.endTag(null, "WIFI_FILTER_SWITCH");
                        xmlSerializer.startTag(null,"WIFI_FILTER_SSID");
                        xmlSerializer.text(ssidStringEditText.getText().toString());
                        xmlSerializer.endTag(null, "WIFI_FILTER_SSID");
                        xmlSerializer.endDocument();
                        xmlSerializer.flush();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
```



### BSP端

Android BSP主要負責就是在Scan AP前，先去Parsing
 */storage/emulator/0/wifi_filter/wifi_filter_setting.xml*
獲得WIFI_FILTER_SWITCH和WIFI_FILTER_SSID後，就可以完成Wifi AP過濾的功能

```java
diff --git a/packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java b/packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java
index 1e9a6bb..aef265c 100644
--- a/packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java
+++ b/packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java
@@ -83,6 +83,15 @@ import java.util.List;
 
 import static android.os.UserManager.DISALLOW_CONFIG_WIFI;
 
+import java.io.File;
+import android.os.Environment;
+import org.xmlpull.v1.XmlPullParser;
+import org.xmlpull.v1.XmlPullParserException;
+import java.io.FileInputStream;
+import java.io.FileNotFoundException;
+import java.io.IOException;
+
+import android.util.Xml;
 /**
  * Two types of UI are provided here.
  *
@@ -649,6 +658,51 @@ public class WifiSettings extends RestrictedSettingsFragment
     public synchronized void onAccessPointsChanged() {
         // Safeguard from some delayed event handling
         if (getActivity() == null) return;
+        
+        String wifiEnableSwith="";
+        String ssid="";
+        
+        File sdCard = Environment.getExternalStorageDirectory();
+        File dir = new File (sdCard.getAbsolutePath() + "/wifi_filter/");
+        File file = new File(dir, "wifi_filter_setting.xml");
+        if(dir.exists() == true)
+        {
+        	try {
+                FileInputStream fis = new FileInputStream(file);
+                XmlPullParser pullParser = Xml.newPullParser();
+                pullParser.setInput(fis, "UTF-8");
+                int fileFlag = pullParser.getEventType();
+
+                while(fileFlag != XmlPullParser.END_DOCUMENT){
+                    String nodeName = pullParser.getName();
+                    switch (fileFlag) {
+                        case XmlPullParser.START_TAG:
+                            if(nodeName.equals("WIFI_FILTER_SWITCH")){
+                                wifiEnableSwith = pullParser.nextText();
+                                Log.d(TAG, "WIFI_FILTER_SWITCH = " + wifiEnableSwith);
+
+                            }
+                            if(nodeName.equals("WIFI_FILTER_SSID")){
+                                ssid = pullParser.nextText();
+                                Log.d(TAG, "WIFI_FILTER_SSID = " + ssid);
+                            }
+
+                            break;
+                        case XmlPullParser.END_TAG:
+                        default:
+                            break;
+                    }
+
+                    // Get the next event type
+                    fileFlag = pullParser.next();
+                }
+                
+                
+        	}catch(XmlPullParserException | IOException e){
+        		e.printStackTrace();
+        	}
+        }
+        
         if (isUiRestricted()) {
             if (!isUiRestrictedByOnlyAdmin()) {
                 addMessagePreference(R.string.wifi_empty_list_user_restricted);
@@ -668,6 +722,12 @@ public class WifiSettings extends RestrictedSettingsFragment
                 int index = 0;
                 cacheRemoveAllPrefs(getPreferenceScreen());
                 for (AccessPoint accessPoint : accessPoints) {
+                	if(TextUtils.isEmpty(wifiEnableSwith) == false && TextUtils.isEmpty(ssid) == false && wifiEnableSwith.equals("1") == true) {
+                		if(accessPoint.getSsidStr().equals(ssid) == false) {
+                			continue;
+                		}
+                	}
+                	
                     // Ignore access points that are out of range.
                     if (accessPoint.getLevel() != -1) {
                         String key = accessPoint.getBssid();
```

### 運行結果

BSP重新燒錄後和app安裝完後，要先去設定->應用程式下把權限給打開

![image-20191217105825664]({{site.baseurl}}/img/image-20191217105825664.png)

首先看正常環境下的wifi訊號

![image-20191217105958330]({{site.baseurl}}/img/image-20191217105958330.png)

接者打開app，設定只想顯示的wifi AP名稱，然後切Enable
![image-20191217110115614]({{site.baseurl}}/img/image-20191217110115614.png)

再回到Wifi頁面下看結果，果然只顯示設定的AP
![image-20191217110218675]({{site.baseurl}}/img/image-20191217110218675.png)