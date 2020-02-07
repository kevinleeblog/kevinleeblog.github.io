---
layout: post
title: "做一個在Android只列出想要的Wi-Fi AP(二) - 使用SharedPreference"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design,Job_Logging]
subtitle: 
visualworkflow: true
---

繼續上篇的任務
[做一個在Android只列出想要的Wi-Fi AP(一)](/2019/12/05/android-bsp-wi-fi-modify)
接下來就是要做一個啟動filter的開關，以及想要出現的Wifi SSID name

> 以下是一個失敗的案例，主要是Android在5或以後的版本認為這樣的做法有安全性的問題，所以強制禁止，使用以下app的程式會出現SecurityException的crash

我的構想是把這些UI的開關設計成app，然後在Android BSP的Wifi Setting下增加code去加一個是否啟用filter的判斷與要固定的ssid，這要改掉上篇是把AP變成Hard code給修改掉。

Android要讓兩個獨立的app之間可以共享Resource，我這邊是使用SharedPreference
想法就是BSP這邊會去判斷兩個字串**WIFI_FILTER_SWITCH**和**WIFI_FILTER_SSID**是否有內容，而這兩個字串是由app去修改而BSP負責讀內容，再依照內容去做動作。

packages/apps/Settings/src/com/android/settings/wifi/WifiSettings.java

```java
    /**
     * Shows the latest access points available with supplemental information like
     * the strength of network and the security for it.
     */
    @Override
    public synchronized void onAccessPointsChanged() {
        String wifiFilterSwitch = null;
        String wifiFilterSSID = null;

        try {
        	Context appContext = getActivity().createPackageContext("com.protech.os.wififilter",getActivity().CONTEXT_INCLUDE_CODE|getActivity().CONTEXT_IGNORE_SECURITY);
        	SharedPreferences myPrefs = appContext.getSharedPreferences("wifi_filter_setting",0);
            wifiFilterSwitch = myPrefs.getString("WIFI_FILTER_SWITCH",null);
            wifiFilterSSID = myPrefs.getString("WIFI_FILTER_SSID",null);
        }catch (NameNotFoundException e) {
        	Log.e(TAG, "NameNotFoundException for wififilter");
        }
        final int wifiState = mWifiManager.getWifiState();
        switch (wifiState) {
            case WifiManager.WIFI_STATE_ENABLED:
                // AccessPoints are automatically sorted with TreeSet.
                final Collection<AccessPoint> accessPoints =
                        mWifiTracker.getAccessPoints();
            
                for (AccessPoint accessPoint : accessPoints) {
                	if(TextUtils.isEmpty(wifiFilterSwitch) == false && TextUtils.isEmpty(wifiFilterSSID) == false && wifiFilterSwitch.equals("1") == true)
                	{
                		if(accessPoint.getSsidStr().equals(wifiFilterSSID) == false)
                			continue;
                	}
```

#### BSP程式解說

使用SharedPreferences是把這些設定值變成檔案的方式存在自己app目錄內的shared_prefs內的xml，*/data/data/YOUR_PACKAGE_NAME/shared_prefs/YOUR_PREFS_NAME.xml*

想讓另外一個app可以存取到另外一個app的設定檔
我是使用createPackageContext先抓取app的package name轉後得到context，再透過那個context得到app目錄下設定檔的內容，聽起來很合理。

#### APP程式解說

APP的UI我這樣設計
![image-20191216102706198]({{site.baseurl}}/img/image-20191216102706198.png)

當Filter的Switch是在disable或Enable其實就是把兩字串寫到wifi_filter_setting.xml

然後BSP在讀取這個檔案來做動作。

app端程式

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        myPrefs = getApplicationContext().getSharedPreferences("wifi_filter_setting" , getApplicationContext().MODE_WORLD_READABLE);
        //myPrefs = PreferenceManager.getDefaultSharedPreferences(getApplicationContext());
        wifiFilterSwitch = myPrefs.getString("WIFI_FILTER_SWITCH",null);
        wifiFilterSSID = myPrefs.getString("WIFI_FILTER_SSID",null);

        wifiFilterEnableSwitch = (Switch)findViewById(R.id.wifiFilterEnableSwitch);
        ssidStringEditText=(EditText)findViewById(R.id.ssidStringEditText);

        // First time initialize SharePreference
        if(wifiFilterSwitch == null || wifiFilterSSID == null)
        {
            SharedPreferences.Editor editor = myPrefs.edit();
            editor.putString("WIFI_FILTER_SWITCH","0");
            editor.putString("WIFI_FILTER_SSID","");
            editor.apply();

            wifiFilterSwitch = myPrefs.getString("WIFI_FILTER_SWITCH",null);
            wifiFilterSSID = myPrefs.getString("WIFI_FILTER_SSID",null);
        }


        if(wifiFilterSwitch.equals("0") == true) {
            wifiFilterEnableSwitch.setChecked(false);
            ssidStringEditText.setText(wifiFilterSSID);
        }
        else {
            wifiFilterEnableSwitch.setChecked(true);
            ssidStringEditText.setFocusable(false);
            ssidStringEditText.setFocusableInTouchMode(false);
            ssidStringEditText.setText(wifiFilterSSID);
        }

        wifiFilterEnableSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {

                SharedPreferences.Editor editor = myPrefs.edit();

                if(isChecked == true)
                {
                    ssidStringEditText.setFocusable(false);
                    ssidStringEditText.setFocusableInTouchMode(false);

                    editor.putString("WIFI_FILTER_SWITCH","1");
                    editor.putString("WIFI_FILTER_SSID",ssidStringEditText.getText().toString());
                    editor.apply();
                }
                else
                {
                    ssidStringEditText.setFocusable(true);
                    ssidStringEditText.setFocusableInTouchMode(true);
                    ssidStringEditText.requestFocus();
                    editor.putString("WIFI_FILTER_SWITCH","0");
                    editor.apply();
                }
            }
        });

    }
```

