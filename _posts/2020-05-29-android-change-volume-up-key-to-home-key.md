---
layout: post
title: "Android把音量鍵換成Home key"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle: Android 9
visualworkflow: true
---

剛接到這個需求時有點傻眼，就是不確定VOL_UP是不是硬體內建的，要改成Home Key不確定是不是要改很多地方，上網詢問古先生無意間又注意到Android系統從Kernel那邊接收到已經mapping後的按鍵訊號（透過/dev/input/eventX發送給Android）會在做一次按鍵remapping。
所以想法很簡單，就是當按下音量鍵時在Kernel端時還是被視為音量鍵，但當傳送給Android時要remapping成Home Key

 Android這個key mapping其實是做成一個檔案被稱為「Key Layout Files」
每當系統重新開機會依序去以下目錄搜尋Key Layout Files

![image-20200529101245805]({{site.baseurl}}/img/image-20200529101245805.png)

但也不是說只要隨便丟一個任意的Key Layout Files，Android就照單全收唷！！
以我要改的Volume up key為例

先使用adb shell進到Android Terminal，打入以下指令

```
$ cat /proc/bus/input/devices
...
I: Bus=0019 Vendor=0001 Product=0001 Version=0100
N: Name="soc:gpio_keys"
P: Phys=gpio-keys/input0
S: Sysfs=/devices/platform/soc/soc:gpio_keys/input/input2
U: Uniq=
H: Handlers=event2 cpufreq 
B: PROP=0
B: EV=3
B: KEY=4000000000000000 0 0 0 0 0 0 0 0 0 8000000000000 0
...
```

其中N: Name="soc:gpio_keys"內的名字要和Key Layout Files一樣，才會吃到這個檔案
不然預設都是會吃Generic.kl
理論上我應該會吃gpio_keys.kl檔，但猜測name為soc::gpio_keys，所以這邊我一直吃不到

至於怎麼確認Volume up key是來自"soc:gpio_keys"？
打入如下命令後，按下Volume up key，就可得知 key event是來自/dev/input/event2就是"soc:gpio_keys"

```
msm8953_64:/ # getevent                                                        
add device 1: /dev/input/event4
  name:     "msm8953-snd-card-mtp Button Jack"
add device 2: /dev/input/event3
  name:     "msm8953-snd-card-mtp Headset Jack"
could not get driver version for /dev/input/mice, Not a typewriter
add device 3: /dev/input/event0
  name:     "qpnp_pon"
add device 4: /dev/input/event1
  name:     "ilitek_ts"
could not get driver version for /dev/input/mouse0, Not a typewriter
add device 5: /dev/input/event2
  name:     "soc:gpio_keys"
  
/dev/input/event2: 0001 0073 00000001
/dev/input/event2: 0000 0000 00000000
/dev/input/event2: 0001 0073 00000000
/dev/input/event2: 0000 0000 00000000
```

另外就是要如何確認"soc:gpio_keys"是吃那個Key Layout Files?

```
$ dumpsys input
...
    5: soc:gpio_keys
      Classes: 0x00000001
      Path: /dev/input/event2
      Enabled: true
      Descriptor: 485d69228e24f5e46da1598745890b214130dbc4
      Location: gpio-keys/input0
      ControllerNumber: 0
      UniqueId: 
      Identifier: bus=0x0019, vendor=0x0001, product=0x0001, version=0x0100
      KeyLayoutFile: /system/usr/keylayout/Generic.kl
      KeyCharacterMapFile: /system/usr/keychars/Generic.kcm
      ConfigurationFile: 
      HaveKeyboardLayoutOverlay: false
...
```

可以確認"soc:gpio_keys"的Key Layout Files是*KeyLayoutFile: /system/usr/keylayout/Generic.kl*

接下來就是探討為何Name為這麼奇怪是"soc:gpio_keys"？

*kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-mtp.dtsi*

```
161 &soc {
162         gpio_keys {
163                 compatible = "gpio-keys";
164                 input-name = "gpio-keys";
165                 pinctrl-names = "default";
166                 pinctrl-0 = <&gpio_key_active>;
......
187 
188                 vol_up {
189                         label = "volume_up";
190                         gpios = <&tlmm 85 0x1>;
191                         linux,input-type = <1>;
192                         linux,code = <115>;
193                         debounce-interval = <15>;
194                         linux,can-disable;
195                         gpio-key,wakeup;
196                 };
197         };
198 };
```

推測"soc:gpio_keys"來自於devicetree中的soc下的gpio_keys，但要如何改？

經過反覆推敲後，認為應是soc->gpio_keys缺少name
所以要加上label並命為"gpio-keys"

*kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953-mtp.dtsi*

```
161 &soc {
162         gpio_keys {
163                 compatible = "gpio-keys";
164                 input-name = "gpio-keys";
165                 label = "gpio-keys";
```

此時修改gpio-keys.kl並把key <115>的VOLUME_UP改成HOME

```
key 115   HOME
key 114   VOLUME_DOWN
......
```

為何是key 115?因為device tree的linux,code = <115>

重新編譯後，按下Volume up就會變成Home Key了