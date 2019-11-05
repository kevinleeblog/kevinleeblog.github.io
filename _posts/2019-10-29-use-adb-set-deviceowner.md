---
layout: post
title: "adb設置DeviceOwner發生問題"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design, Job_Logging]
subtitle:
visualworkflow: true
---

### 為何做

最近有一個日本客戶在評估我們產品，就提出了一個在Quectel SC60(Android 7.1.2)無法設置DeviceOwner的問題
操作方式是使用adb去安裝apk後，然後設置apk權限為DeviceOwner
但是發生"*Not allowed to set the device owner because there are already some accounts on the device*"

```
$ adb install TestDPC_6102.apk
$ adb shell dpm set-device-owner com.afwsamples.testdpc/.DeviceAdminReceiver
java.lang.IllegalStateException: Not allowed to set the device owner because there are already some accounts on the device
	at android.os.Parcel.readException(Parcel.java:1692)
	at android.os.Parcel.readException(Parcel.java:1637)
	at android.app.admin.IDevicePolicyManager$Stub$Proxy.setDeviceOwner(IDevicePolicyManager.java:4779)
	at com.android.commands.dpm.Dpm.runSetDeviceOwner(Dpm.java:148)
	at com.android.commands.dpm.Dpm.onRun(Dpm.java:96)
	at com.android.internal.os.BaseCommand.run(BaseCommand.java:51)
	at com.android.commands.dpm.Dpm.main(Dpm.java:41)
	at com.android.internal.os.RuntimeInit.nativeFinishInit(Native Method)
	at com.android.internal.os.RuntimeInit.main(RuntimeInit.java:262)
```



### Download testDPC

TestDPC為Google提供的apk，可自行自google取得
https://github.com/googlesamples/android-testdpc/releases/tag/6.1.2-preview
使用testDPC驗證Device Administrator
https://source.android.com/devices/tech/admin/testing-setup

### 如何Debug

根據錯誤回報"Not allowed to set the device owner because there are already some accounts on the device"
首先在Android內
Setting->Accounts,確認是否把所有的帳戶給清掉
![IWGOdtY]({{site.baseurl}}/img/IWGOdtY.png)

使用adb檢查

```
$ adb shell dumpsys account
User UserInfo{0:擁有者:13}:
  Accounts: 1
    Account {name=PHONE, type=com.android.localphone}
  
  AccountId, Action_Type, timestamp, UID, TableName, Key
  Accounts History
  1,action_account_add,1970-01-03 03:56:09,1000,accounts,0
  
  Active Sessions: 0
  
  RegisteredServicesCache: 5 services
    ServiceInfo: AuthenticatorDescription {type=com.android.sim}, ComponentInfo{com.qualcomm.simcontacts/com.qualcomm.simcontacts.SimAuthenticateService}, uid 1000
    ServiceInfo: AuthenticatorDescription {type=com.android.exchange}, ComponentInfo{com.android.email/com.android.email.service.EasAuthenticatorService}, uid 10054
    ServiceInfo: AuthenticatorDescription {type=com.android.localphone}, ComponentInfo{com.qualcomm.simcontacts/com.qualcomm.simcontacts.PhoneAuthenticateService}, uid 1000
    ServiceInfo: AuthenticatorDescription {type=com.android.email.pop3}, ComponentInfo{com.android.email/com.android.email.service.Pop3AuthenticatorService}, uid 10054
    ServiceInfo: AuthenticatorDescription {type=com.android.email.legacy.imap}, ComponentInfo{com.android.email/com.android.email.service.LegacyImapAuthenticatorService}, uid 10054
```

發現竟然有一個隱藏的account
name = PHONE
type = com.android.localphone

解決這個問題，我大概想了三種方式處理

#### 方式1

Download testDPC Source Code，然後在AndroidManifest.xml內設置
`android:testOnly="true"`
![J6nnKz3]({{site.baseurl}}/img/J6nnKz3.png)

然後執行安裝並設置DeviceOwner

```
$ adb install TestDPC_debug.apk
$ adb shell dpm set-device-owner com.afwsamples.testdpc/.DeviceAdminReceiver
```

#### 方式2

使用adb先把account先隱藏起來，然後在設置DeviceOwner
先前使用dumpsys account可知道Account是
`Account {name=PHONE, type=com.android.localphone}`
在由下方RegisteredServicesCaches可得知這個Account是由那一個Package所註冊
所以是com.qualcomm.simcontacts這個package

```
ServiceInfo: AuthenticatorDescription {type=com.android.localphone}, ComponentInfo{com.qualcomm.simcontacts/com.qualcomm.simcontacts.PhoneAuthenticateService}, uid 1000
```

然後把這個Package給隱藏起來，然後在設置DeviceOwner權限

```
$ adb shell pm hide com.qualcomm.simcontacts
$ adb install TestDPC_6102.apk
$ adb shell dpm set-device-owner com.afwsamples.testdpc/.DeviceAdminReceiver
```

但在我使用的Android版本在hide account時會出現Shell權限不夠的問題
`Neither user 2000 nor current process has android.permission.MANAGE_USERS`
解決方式是在Android BSP中加入權限給Shell
在frameworks/base/packages/Shell/AndroidManifest.xml
加上
<uses-permission android:name="android.permission.MANAGE_USERS" />
重新編譯後，Shell就有hide account的能力了

#### 方式3

這是我覺得最好的方式，就是在Android BSP中就不安裝會註冊account的apk
可先搜尋一下com.qualcomm.simcontacts的apk存放位置在那

```
$ adb pm list package -f com.qualcomm.simcontacts
```

可得知apk是放在/system/app/SimContacts/SimContacts.apk
然後就在Android BSP內把此Package設成不安裝
在vendor/qcom/proprietary/telephony-apps/SimContacts/Android.mk
把Enable_SimContacts:=yes
改成
Enable_SimContacts:=**no**
然後執行Clean Build
之後的rom燒到Platform上account就完全清空了
就可直接安裝並設置DeviceOwner權限了