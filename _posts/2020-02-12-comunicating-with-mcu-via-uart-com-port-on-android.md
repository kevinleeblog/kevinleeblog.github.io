---
layout: post
title: "Comunicating with MCU via Uart com port on Android"
auther: Kevin Lee
category: project1
tags: [Android]
subtitle: 
visualworkflow: true
---

### 為何做？

Android如果想要擴充額外的功能，去做到Android本身沒法達到的事情，就必須依靠MCU，所以Android和MCU之間的通訊可以透過Uart或是USB來完成。

現在公司有一應用是需要有額外GPIO在Android上，所以Android使用外接USB轉RS232來產生Uart Com Port來和MCU之間通訊並使用MCU上面的GPIO來達到擴充Android的功能

大概的接線如下圖

![IMG_0059]({{site.baseurl}}/img/IMG_0059.jpg)

### 如何做？

這邊我直接使用PL2303G提供的[Android USB Host API SDK](http://www.prolific.com.tw/UserFiles/files/PL2303G_Android_v1000_20180827.zip)來改成我要的功能

![image-20200212154738156]({{site.baseurl}}/img/image-20200212154738156.png)

SDK本身是使用USB HID的方式和PL2303G溝通，所以不需要選擇RS232 COM Port

以下就是如何使用這個SDK的方式

打開Android Studio創建新專案 -> Empty Activity

##### 導入PL2303G Library

複製SDK內的*pl2303g_driver.jar*至新專案的目錄libs內
![image-20200212155649682]({{site.baseurl}}/img/image-20200212155649682.png)



回到專案內，選擇剛剛複製的library
右鍵，選擇Add As Library

![image-20200212160009427]({{site.baseurl}}/img/image-20200212160009427.png)



引用Library

```java
import tw.com.prolific.driver.pl2303g.PL2303GDriver;
```



##### Android使用USB前要先修改manifest

根據Android USB開發文件https://developer.android.com/guide/topics/connectivity/usb/host

![image-20200213093200846]({{site.baseurl}}/img/image-20200213093200846.png)

*AndroidManifest.xml*

```xml
<manifest ...>
    <uses-feature android:name="android.hardware.usb.host" />
    <uses-sdk android:minSdkVersion="12" />
    ...
    <application>
        <activity ...>
            ...
            <intent-filter>
                <action android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED" />
            </intent-filter>

            <meta-data android:name="android.hardware.usb.action.USB_DEVICE_ATTACHED"
                android:resource="@xml/device_filter" />
        </activity>
    </application>
</manifest>
```

*res/xml/device_filter.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
		<usb-device vendor-id="1659" product-id="9123" />
		<usb-device vendor-id="1659" product-id="9139" />
		<usb-device vendor-id="1659" product-id="9155" />
		<usb-device vendor-id="1659" product-id="9171" />
		<usb-device vendor-id="1659" product-id="9187" />
		<usb-device vendor-id="1659" product-id="8992" />
		<usb-device vendor-id="1659" product-id="8993" />
		<usb-device vendor-id="1659" product-id="8994" />
		<usb-device vendor-id="1659" product-id="8995" />
		<usb-device vendor-id="1659" product-id="9008" />
		<usb-device vendor-id="1659" product-id="9009" />
<!-- usb-device vendor-id="1659=0x067B" product-id="8963=0x23A3" -->
<!-- usb-device vendor-id="1659=0x067B" product-id="8964=0x23B3" -->
<!-- usb-device vendor-id="1659=0x067B" product-id="41216=0x23C3" -->
</resources>
```

##### UI設計

![image-20200213094120978]({{site.baseurl}}/img/image-20200213094221653.png)

##### 主程式修改

```java
package com.protech.os.sc600cradletestprogram;

import tw.com.prolific.driver.pl2303g.PL2303GDriver;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Context;
import android.hardware.usb.UsbManager;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;


public class MainActivity extends AppCompatActivity {
    String TAG = "kevin_test";
    private static final boolean SHOW_DEBUG = true;
    private static final String NULL = null;

    private Button btGetSN;
    private TextView tvShowSN;

    PL2303GDriver mSerial;

    private static final String ACTION_USB_PERMISSION = "com.prolific.PL2303Gsimpletest.USB_PERMISSION";

    public static byte[] hexStringToByteArray(String s) {
        int len = s.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            data[i / 2] = (byte) ((Character.digit(s.charAt(i), 16) << 4)
                    + Character.digit(s.charAt(i+1), 16));
        }
        return data;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btGetSN = (Button) findViewById(R.id.btn_GetSN);
        tvShowSN = (TextView) findViewById(R.id.text_ShowSN);
        tvShowSN.setText("");
        btGetSN.setOnClickListener(new Button.OnClickListener() {
            public void onClick(View v) {
                byte[] sendBuf = new byte[4];
                byte[] readBuf = new byte[20];
                sendBuf=hexStringToByteArray("1455bb6c");

                if(null==mSerial)
                    return;

                if(!mSerial.isConnected())
                    return;

                int nBytes = mSerial.write(sendBuf, sendBuf.length);
                if(nBytes < 0)
                {
                    Log.d(TAG, "Fail to write="+nBytes);
                    return;
                }

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                nBytes = mSerial.read(readBuf);
                if(nBytes<0) {
                    Log.d(TAG, "Fail to bulkTransfer(read data)");
                    return;
                }

                if (nBytes > 0) {
                    if (SHOW_DEBUG) {
                        Log.d(TAG, "read len : " + nBytes);
                    }

                    Log.d(TAG,"nBytes=" + nBytes);

                    String str = new String(readBuf);

                    tvShowSN.setText(str);
                }
                //ShowPL2303G_SerialNmber();
            }
        });

        // get service
        mSerial = new PL2303GDriver((UsbManager) getSystemService(Context.USB_SERVICE),
                this, ACTION_USB_PERMISSION);


        // check USB host function.
        if (!mSerial.PL2303USBFeatureSupported()) {

            Toast.makeText(this, "No Support USB host API", Toast.LENGTH_SHORT)
                    .show();

            Log.d(TAG, "No Support USB host API");

            mSerial = null;
            btGetSN.setEnabled(false);
            tvShowSN.setText("empty");
        }
        else
        {
            if( !mSerial.enumerate() ) {
                Toast.makeText(this, "no more devices found", Toast.LENGTH_SHORT).show();
                btGetSN.setEnabled(false);
                tvShowSN.setText("empty");
            }
            else
            {
                try {
                    Thread.sleep(1500);
                    openUsbSerial();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        Log.d(TAG, "Leave onCreate");
    }

    private void openUsbSerial() {
        Log.d(TAG, "Enter  openUsbSerial");

        if(mSerial==null) {

            Log.d(TAG, "No mSerial");
            return;

        }


        if (mSerial.isConnected()) {
            if (SHOW_DEBUG) {
                Log.d(TAG, "openUsbSerial : isConnected ");
            }

            if (!mSerial.InitByBaudRate(PL2303GDriver.BaudRate.B19200,700)) {
                if(!mSerial.PL2303Device_IsHasPermission()) {
                    Toast.makeText(this, "cannot open, maybe no permission", Toast.LENGTH_SHORT).show();
                }

                if(mSerial.PL2303Device_IsHasPermission() && (!mSerial.PL2303Device_IsSupportChip())) {
                    Toast.makeText(this, "cannot open, maybe this chip has no support, please use PL2303G chip.", Toast.LENGTH_SHORT).show();
                    Log.d(TAG, "cannot open, maybe this chip has no support, please use PL2303G chip.");
                }
            } else {

                Toast.makeText(this, "connected : OK" , Toast.LENGTH_SHORT).show();
                Log.d(TAG, "connected : OK");
                Log.d(TAG, "Exit  openUsbSerial");

            }
            btGetSN.setEnabled(true);
        }//isConnected
        else {
            Toast.makeText(this, "Connected failed, Please plug in PL2303 cable again!" , Toast.LENGTH_SHORT).show();
            Log.d(TAG, "connected failed, Please plug in PL2303 cable again!");
        }
    }
}

```

