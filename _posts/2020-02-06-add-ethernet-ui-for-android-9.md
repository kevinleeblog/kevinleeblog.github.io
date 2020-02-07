---
layout: post
title: "Add Ethernet UI for Android 9"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design,Job_Logging]
subtitle: Android 9
visualworkflow: true

---

### 為何做？

上一篇把USB LAN Driver移植到Android 9之後，雖然可以走有線網路，但是卻無法得知該有線網路的資訊，例如IP address等等，所以就想說增加一個Ethernet UI畫面至設定(System Setting)那邊。

### 如何做？

這邊有參考一些網路上的文章，但我有修改一下，像是把設定靜態IP的功能給拿掉，只單純顯示DHCP的網路資訊。

處理方式大致分為以下幾個部分

##### Ethernet UI

設計UI的部分並加到系統設定畫面下
*packages/apps/Settings/res/values/strings.xml*

```xml
@@ -3411,6 +3411,50 @@
     <!-- Message of the error message shown when error happens during erase eSIM data [CHAR LIMIT=NONE] -->
     <string name="reset_esim_error_msg">The eSIMs can\u2019tt be reset due to an error.</string>
 
+    <!-- Ethernet Settings -->
+    <string name="ethernet_settings_title">Ethernet</string>
+    <string name="ethernet_netmask_hint" translatable="false"> 255.255.255.0</string>
+    <!--string name="ethernet_info_getting">"getting IP info..."</string-->
+    <string name="ethernet_settings">Ethernet</string>
+    <string name="ethernet_connect">Connect</string>
+    <string name="ethernet_cancel">Cancel</string>
+    <!--Wireless controls screen, settings summary for the item tot ake you to the ethernet settings screen -->
+    <string name="ethernet_settings_summary">Manager ethernet</string>
+    <!-- ethernet hw address  -->
+    <string name="ethernet_hw_addr">MAC</string>
+    <!-- ethernet ip address  -->
+    <string name="ethernet_ip_addr">IP address</string>
+    <!-- ethernet netmask  -->
+    <string name="ethernet_netmask">netmask</string>
+    <!-- ethernet gateway  -->
+    <string name="ethernet_gateway">gateway</string>
+    <!-- ethernet dns1  -->
+    <string name="ethernet_dns1">dns1</string>
+    <!-- ethernet dns2 -->
+       <string name="ethernet_dns2">dns2</string>
+       <string name="category_ethernet">Static IP Setttings</string>
+       <string name="usedhcp">dhcp</string>
+       <string name="usestatic">static</string>
+       <string name="ethernet_use_static_ip">Use static IP</string>
+       <string name="ethernet_ip_address">IP address</string>
+       <string name="staticip_save">Save</string>
+       <string name="staticip_cancel">Cancel</string>
+       <string name="str_ok">OK</string>
+       <string name="str_cancel">Cancel</string>
+       <string name="str_about">Important</string>
+       <string name="str_mesg">Whether save Settings?</string>
+       <string name="save_failed">Save failed!</string>
+       <string name="ethernet_ip_settings_invalid_ip">Please type a valid IP address.</string>
+       <string name="eth_ip_settings_please_complete_settings">Please give complete static IP settings!</string>
+       <string name="ethernet_quick_toggle_title">Ethernet</string>
+       <!-- Ethernet settings check box summary for turning on ethernet -->
+       <string name="ethernet_quick_toggle_summary_off">Ethernet is disabled</string>
+       <!--Used as title on second screen after selecting Ethernet settings -->
+       <string name="ethernet_quick_toggle_summary_on">Ethernet is enabled</string>
+       <!--Used as title on second screen after selecting Ethernet settings -->
+       <string name="ethernet_mode_title">Ethernet Ip mode</string>
+       <string name="ethernet_info_getting">"getting IP info..."</string>
+
     <!-- Master Clear -->
     <!-- Button title to factory data reset the entire device -->
     <string name="master_clear_title">Erase all data (factory reset)</string>
```

新增檔案要顯示Ethernet那些資訊，下面是只要IP address, Netmask和Gateway

*packages/apps/Settings/res/xml/ethernet_settings.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:settings="http://schemas.android.com/apk/res/com.android.settings"
    android:title="@string/ethernet_settings" >

    <!-- $_rbox_$_modify_$_lijiehong: change to support bluetooth checkbox -->
    
    <!-- ethernet ip address -->
    <Preference
        style="?android:preferenceInformationStyle"
        android:key="ethernet_ip_addr"
        android:summary="@string/device_info_default"
        android:title="@string/ethernet_ip_addr" />

    <!-- ethernet netmask -->
    <Preference
        style="?android:preferenceInformationStyle"
        android:key="ethernet_netmask"
        android:summary="@string/device_info_default"
        android:title="@string/ethernet_netmask" />

    <!-- ethernet gateway -->
    <Preference
        style="?android:preferenceInformationStyle"
        android:key="ethernet_gateway"
        android:summary="@string/device_info_default"
        android:title="@string/ethernet_gateway" />

</PreferenceScreen>
```



新增Ethernet Icon

*packages/apps/Settings/res/drawable/ic_ethernet.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
  Copyright (C) 2016 The Android Open Source Project

  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License
  -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="24dp"
        android:height="24dp"
        android:viewportWidth="24.0"
        android:viewportHeight="24.0"
        android:tint="?android:attr/colorControlNormal">
    <path
        android:fillColor="#FF000000"
        android:pathData="M7.77,6.76L6.23,5.48 0.82,12l5.41,6.52 1.54,-1.28L3.42,12l4.35,-5.24zM7,13h2v-2L7,11v2zM17,11h-2v2h2v-2zM11,13h2v-2h-2v2zM17.77,5.48l-1.54,1.28L20.58,12l-4.35,5.24 1.54,1.28L23.18,12l-5.41,-6.52z"/>
</vector>
```

##### 程式功能

新增程式去獲取Ethernet Information

*packages/apps/Settings/src/com/android/settings/ethernet/EthernetSettings.java*

```java
/*
 * Copyright (C) 2009 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.android.settings;

import com.android.settings.R;

import android.app.Activity;
import android.app.AlertDialog;
import android.app.Dialog;
import android.app.admin.DevicePolicyManager;
import android.content.ActivityNotFoundException;
import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.content.res.Resources;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.os.SystemProperties;
import android.os.UserHandle;
import android.os.UserManager;
import android.preference.CheckBoxPreference;
import android.preference.Preference.OnPreferenceChangeListener;
import android.preference.PreferenceScreen;
import android.support.v14.preference.SwitchPreference;
import android.support.v7.preference.ListPreference;
import android.support.v7.preference.Preference;
import android.provider.SearchIndexableResource;
import android.provider.Settings;
import android.telephony.TelephonyManager;
import android.text.TextUtils;
import android.util.Log;
import android.content.Intent;

import java.io.File;
import java.io.FileDescriptor;
import java.io.File;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;

import java.util.regex.Pattern;
import java.lang.Integer;
import java.net.InetAddress;
import java.net.Inet4Address;
import java.util.Iterator;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;

import android.preference.Preference.OnPreferenceChangeListener;
import android.preference.Preference.OnPreferenceClickListener;

import com.android.settings.SettingsPreferenceFragment.SettingsDialogFragment;

/*for 5.0*/
import android.net.EthernetManager;
import android.net.IpConfiguration;
import android.net.IpConfiguration.IpAssignment;
import android.net.IpConfiguration.ProxySettings;
import android.net.wifi.SupplicantState;
import android.net.wifi.WifiInfo;
import android.net.wifi.WifiManager;
import android.net.StaticIpConfiguration;
import android.net.NetworkUtils;
import android.net.LinkAddress;
import android.net.LinkProperties;
import android.widget.Toast;
//import android.preference.ListPreference;
//import com.android.internal.logging.MetricsProto.MetricsEvent;
import com.android.internal.logging.nano.MetricsProto.MetricsEvent;

import java.util.Enumeration;
import java.net.NetworkInterface;
import java.util.Collections;
import java.net.InterfaceAddress;

public class EthernetSettings extends SettingsPreferenceFragment {
    private static final String TAG = "EthernetSettings";

    public enum ETHERNET_STATE{
        ETHER_STATE_DISCONNECTED,
        ETHER_STATE_CONNECTING,
        ETHER_STATE_CONNECTED
    }

    private static final String KEY_ETH_IP_ADDRESS = "ethernet_ip_addr";
    private static final String KEY_ETH_HW_ADDRESS = "ethernet_hw_addr";
    private static final String KEY_ETH_NET_MASK = "ethernet_netmask";
    private static final String KEY_ETH_GATEWAY = "ethernet_gateway";

    private static String mEthHwAddress = null;
    private static String mEthIpAddress = null;
    private static String mEthNetmask = null;
    private static String mEthGateway = null;
    private final static String nullIpInfo = "0.0.0.0";

    private final IntentFilter mIntentFilter;
    IpConfiguration mIpConfiguration;
    EthernetManager mEthManager;
    StaticIpConfiguration mStaticIpConfiguration;
    Context mContext;
    private String mIfaceName;
    private long mChangeTime;
    private static final int SHOW_RENAME_DIALOG = 0;
    private static final int ETHER_IFACE_STATE_DOWN = 0;
    private static final int ETHER_IFACE_STATE_UP = 1;

    private static final String FILE = "/sys/class/net/eth0/flags";
    private static final int MSG_GET_ETHERNET_STATE = 0;

    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            if (MSG_GET_ETHERNET_STATE == msg.what) {
                handleEtherStateChange((ETHERNET_STATE) msg.obj);
            }
        }
    };

    @Override
    public int getMetricsCategory() {
        return MetricsEvent.WIFI_TETHER_SETTINGS;
    }

    @Override
    public int getDialogMetricsCategory(int dialogId) {
        switch (dialogId) {
            case SHOW_RENAME_DIALOG:
                return MetricsEvent.WIFI_TETHER_SETTINGS;
            default:
                return 0;
        }
    }


    private final BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            Log.d(TAG,"Receiver Action " + action);
            if (ConnectivityManager.CONNECTIVITY_ACTION.equals(action)) {
                NetworkInfo info = intent.getParcelableExtra(ConnectivityManager.EXTRA_NETWORK_INFO);
                Log.d(TAG, "info ==="+info.toString());
                if (null != info && ConnectivityManager.TYPE_ETHERNET == info.getType()) {
                    long currentTime = System.currentTimeMillis();
                    int delayTime = 0;
                    if (currentTime - mChangeTime < 1000) {
                        delayTime = 2000;
                    }
                    if (NetworkInfo.State.CONNECTED == info.getState()) {
                        handleEtherStateChange(ETHERNET_STATE.ETHER_STATE_CONNECTED, delayTime);
                    } else if(NetworkInfo.State.DISCONNECTED == info.getState()) {
                        handleEtherStateChange(ETHERNET_STATE.ETHER_STATE_DISCONNECTED, delayTime);
                    }
                }
            }
        }
    };

    public EthernetSettings() {
        mIntentFilter = new IntentFilter();
        mIntentFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION);
    }

    private void handleEtherStateChange(ETHERNET_STATE EtherState, long delayMillis) {
        mHandler.removeMessages(MSG_GET_ETHERNET_STATE);
        if (delayMillis > 0) {
            Message msg = new Message();
            msg.what = MSG_GET_ETHERNET_STATE;
            msg.obj = EtherState;
            mHandler.sendMessageDelayed(msg, delayMillis);
        } else {
            handleEtherStateChange(EtherState);
        }
    }

    private void handleEtherStateChange(ETHERNET_STATE EtherState) {
        Log.d(TAG,"curEtherState" + EtherState);

        switch (EtherState) {
            case ETHER_STATE_DISCONNECTED:
                mEthHwAddress = nullIpInfo;
                mEthIpAddress = nullIpInfo;
                mEthNetmask = nullIpInfo;
                mEthGateway = nullIpInfo;
                break;
            case ETHER_STATE_CONNECTING:
                String mStatusString = this.getResources().getString(R.string.ethernet_info_getting);
                mEthHwAddress = mStatusString;
                mEthIpAddress = mStatusString;
                mEthNetmask = mStatusString;
                mEthGateway = mStatusString;
                break;
            case ETHER_STATE_CONNECTED:
                getEthInfo();
                break;
        }

        refreshUI();
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        addPreferencesFromResource(R.xml.ethernet_settings);

        mContext = this.getActivity().getApplicationContext();
        mEthManager = (EthernetManager) getSystemService(Context.ETHERNET_SERVICE);

        if (mEthManager == null) {
            Log.e(TAG, "get ethernet manager failed");
            Toast.makeText(mContext, R.string.disabled_low_ram_device, Toast.LENGTH_SHORT).show();
            finish();
            return;
        }
        String[] ifaces = mEthManager.getAvailableInterfaces();
        if (ifaces.length > 0) {
            mIfaceName = ifaces[0];//"eth0";
	    Log.d(TAG, "get ethernet name:" + mIfaceName);
        }
        if (null == mIfaceName) {
            Log.e(TAG, "get ethernet ifaceName failed");
            Toast.makeText(mContext, R.string.disabled_low_ram_device, Toast.LENGTH_SHORT).show();
            finish();
        }
    }

    @Override
    public void onResume() {
        super.onResume();
        if (null == mIfaceName) {
            return;
        }
        refreshUI();
        mContext.registerReceiver(mReceiver, mIntentFilter);
    }

    @Override
    public void onPause() {
        super.onPause();
        if (null == mIfaceName) {
            return;
        }
        mContext.unregisterReceiver(mReceiver);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        mHandler.removeMessages(MSG_GET_ETHERNET_STATE);
    }

    @Override
    public void onStop() {
        super.onStop();
    }

    private void setStringSummary(String preference, String value) {
        try {
            findPreference(preference).setSummary(value);
        } catch (RuntimeException e) {
            findPreference(preference).setSummary("");
        }
    }

    private void refreshUI() {

        //    setStringSummary(KEY_ETH_HW_ADDRESS,mEthHwAddress);

        setStringSummary(KEY_ETH_IP_ADDRESS, mEthIpAddress);
        setStringSummary(KEY_ETH_NET_MASK, mEthNetmask);
        setStringSummary(KEY_ETH_GATEWAY, mEthGateway);
    }

    public static String calcMaskByPrefixLength(int length) {

        int mask = 0xffffffff << (32 - length);
        int partsNum = 4;
        int bitsOfPart = 8;
        int maskParts[] = new int[partsNum];
        int selector = 0x000000ff;

        for (int i = 0; i < maskParts.length; i++) {
            int pos = maskParts.length - 1 - i;
            maskParts[pos] = (mask >> (i * bitsOfPart)) & selector;
        }

        String result = "";
        result = result + maskParts[0];
        for (int i = 1; i < maskParts.length; i++) {
            result = result + "." + maskParts[i];
        }
        return result;
    }    

    public static String calcSubnetAddress(String ip, String mask) {
        String result = "";
        try {
            // calc sub-net IP
            InetAddress ipAddress = InetAddress.getByName(ip);
            InetAddress maskAddress = InetAddress.getByName(mask);


            byte[] ipRaw = ipAddress.getAddress();
            byte[] maskRaw = maskAddress.getAddress();


            int unsignedByteFilter = 0x000000ff;
            int[] resultRaw = new int[ipRaw.length];
            for (int i = 0; i < resultRaw.length; i++) {
                resultRaw[i] = (ipRaw[i] & maskRaw[i] & unsignedByteFilter);
            }


            // make result string
            result = result + resultRaw[0];
            for (int i = 1; i < resultRaw.length; i++) {
                result = result + "." + resultRaw[i];
            }
        } catch (Exception e) {
            e.printStackTrace();
        }


        return result;
    }

    public void getEthInfoFromDhcp() {
	try{
	    List<NetworkInterface> interfaces = Collections.list(NetworkInterface.getNetworkInterfaces());
	    for(NetworkInterface intf : interfaces){
	    	if(!intf.getName().equalsIgnoreCase(mIfaceName)) continue;

		for(InterfaceAddress ia : intf.getInterfaceAddresses()){
		    if(ia.getAddress().getHostAddress().indexOf(':') < 0){
		        mEthIpAddress = ia.getAddress().getHostAddress();
				mEthNetmask = calcMaskByPrefixLength(ia.getNetworkPrefixLength());
				mEthGateway = calcSubnetAddress(mEthIpAddress,mEthNetmask);
		    	Log.d(TAG, String.format("%s: %s/%s", intf.getDisplayName(), ia.getAddress(), mEthNetmask));
		    }
		}
	    }
	}catch(Exception ignored){
	    mEthIpAddress = nullIpInfo;
	    mEthNetmask = nullIpInfo;
	    mEthGateway = nullIpInfo;
	}
    }

    /*
     * TODO:
     */
    public void getEthInfo() {
        /*
        mEthHwAddress = mEthManager.getEthernetHwaddr(mEthManager.getEthernetIfaceName());
        if (mEthHwAddress == null) mEthHwAddress = nullIpInfo;
        */
        IpAssignment mode = mEthManager.getConfiguration(mIfaceName).getIpAssignment();


        if (mode == IpAssignment.DHCP || mode == IpAssignment.UNASSIGNED) {
        /*
         * getEth from dhcp
        */
            getEthInfoFromDhcp();
        }
    }

    /*
     * tools
    */
    private void log(String s) {
        Log.d(TAG, s);
    }

}
```



*packages/apps/Settings/src/com/android/settings/core/gateway/SettingsGateway.java*

```xml
diff --git a/packages/apps/Settings/src/com/android/settings/core/gateway/SettingsGateway.java b/packages/apps/Settings/src/com/android/settings/core/gateway/SettingsGateway.java
index a8cb61c..e47affb 100644
--- a/packages/apps/Settings/src/com/android/settings/core/gateway/SettingsGateway.java
+++ b/packages/apps/Settings/src/com/android/settings/core/gateway/SettingsGateway.java
@@ -19,6 +19,7 @@ package com.android.settings.core.gateway;
 import com.android.settings.DateTimeSettings;
 import com.android.settings.DeviceAdminSettings;
 import com.android.settings.DisplaySettings;
+import com.android.settings.EthernetSettings;
 import com.android.settings.IccLockSettings;
 import com.android.settings.MasterClear;
 import com.android.settings.PrivacySettings;
@@ -153,6 +154,7 @@ public class SettingsGateway {
             WifiP2pSettings.class.getName(),
             BackgroundCheckSummary.class.getName(),
             VpnSettings.class.getName(),
+           EthernetSettings.class.getName(),
             DateTimeSettings.class.getName(),
             LocaleListEditor.class.getName(),
             AvailableVirtualKeyboardFragment.class.getName(),
```

實際效果～～

![IMG_0004]({{site.baseurl}}/img/IMG_0004.jpg)

