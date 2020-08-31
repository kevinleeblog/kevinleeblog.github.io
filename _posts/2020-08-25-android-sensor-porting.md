---
layout: post
title: "Android dsps Sensor開發過程"
auther: Kevin Lee
category: project1w
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle:
visualworkflow: true
---

最近Android的Ambient Light Sensor從公司板子做好後一直都不會有反應，只有G-Sensor有反應，在其他功能大致上處理好後，開始回頭查到底是什麼問題。

一開始查後發現使用的sensor i2c address是錯的，但換回正確的slave address並重新編譯，既然還是不會動，而且感覺沒有燒進去的感覺，上網問古先生後，終於了解為什麼了

在sensor_def_qcomdev.conf把sensor的位址改對

```xml-dtd
--- a/vendor/qcom/proprietary/sensors/dsps/reg_defaults/sensor_def_qcomdev.conf
+++ b/vendor/qcom/proprietary/sensors/dsps/reg_defaults/sensor_def_qcomdev.conf
@@ -1430,8 +1430,8 @@
 1957 0xFFFF 0x00010001             #gpio1
 1958 0xFFFF 0x00010001             #gpio2
 1959 40 0x00010001                 #sensor_id
-1960 0x48 0x00010001               #i2c_address
-1961 5 0x00010001                  #data_type1
+1960 0x1c 0x00010001               #i2c_address
+1961 0 0x00010001                  #data_type1
 1962 6 0x00010001                  #data_type2
 1963 -1 0x00010001                 #rel_sns_idx
 1964 0 0x00010001                  #sens_default
```

重新編譯並燒到板子裡

但light sensor無反應，檢查機器內檔案是否有更新到

```
$ cd /vendor/etc/sensors
$ cat sensor_def_qcomdev.conf
...
####################################################
###                     ALSPS                    ###
####################################################
# SSI SMGR Cfg 3: [STK LIGHT POLL] AL3320A
1951 0xb0e4f4c92c3e8abc 0x00010001 #UUID
1950 0x534749548d64da24 0x00010001 #UUID
1952 5700 0x00010001               #off_to_idle
1953 10000 0x00010001              #idle_to_ready
1954 4 0x00010001                  #i2c_bus
1955 1040 0x00010001               #reg_group_id
1956 0xFFFF 0x00010001             #cal_grp_id
1957 43 0x00010001                 #gpio1
1958 0xFFFF 0x00010001             #gpio2
1959 40 0x00010001                 #sensor_id
1960 0x1c 0x00010001               #i2c_address
1961 0 0x00010001                  #data_type1
1962 6 0x00010001                  #data_type2
1963 -1 0x00010001                 #rel_sns_idx
1964 0 0x00010001                  #sens_default
1965 0x80 0x00010001               #flags
1985 0 0x00010001                  #device_select
1993 0x93 0x00010001               #vdd
1994 0x02 0x00010001               #vddio
...
```

看起來i2c address的確有更新到0x1c

我用示波器也量測不到0x1c的訊號，只有G-sensor的訊號
問古先生後，原來不用辛苦使用示波器從終端機下command就可以知道sensor有沒有掛載成功

```
1|console:/ # sns_regedit_ssi -r
device platform not detected
detected SNS_PLATFORM_MTP...
Reading existing registry...
SSI Group registry read successful

==========   SMGR Config Group 0   ====================
Version: 1.1
drv_cfg[0]
-----------------------------
item-registry-ID: name: value...
1902-1903: UUID: LIS3DH 14b6faeb-e028-4b3a-bfff-7d04f575ac14
1904: off_to_idle:   100000     1905: idle_to_ready:  20000     1906: i2c_bus:         0x04
1907: reg_group_id:    1000     1908: cal_grp_id:         0     1909: gpio1:         0x002a
1910: gpio2:         0xffff     1911: sensor_id:          0     1912: i2c_address:     0x18
1913: data_type1:         1     1914: data_type2:         0     1915: rel_sns_idx:       -1
1916: sens_default:       1     1917: flags:           0x00
1982: device_select:      0
1982: vdd_rail:    147
1982: vddio_rail:      2
drv_cfg[1]
-----------------------------
item-registry-ID: name: value...
1918-1919: UUID: NULL 00000000-0000-0000-0000-000000000000
1920: off_to_idle:        0     1921: idle_to_ready:      0     1922: i2c_bus:         0x00
1923: reg_group_id:       0     1924: cal_grp_id:         0     1925: gpio1:         0x0000
1926: gpio2:         0x0000     1927: sensor_id:          0     1928: i2c_address:     0x00
1929: data_type1:         0     1930: data_type2:         0     1931: rel_sns_idx:        0
1932: sens_default:       0     1933: flags:           0x00
1983: device_select:      0
1983: vdd_rail:    255
1983: vddio_rail:    255
drv_cfg[2]
-----------------------------
item-registry-ID: name: value...
1934-1935: UUID: NULL 00000000-0000-0000-0000-000000000000
1936: off_to_idle:        0     1937: idle_to_ready:      0     1938: i2c_bus:         0x00
1939: reg_group_id:       0     1940: cal_grp_id:         0     1941: gpio1:         0x0000
1942: gpio2:         0x0000     1943: sensor_id:          0     1944: i2c_address:     0x00
1945: data_type1:         0     1946: data_type2:         0     1947: rel_sns_idx:        0
1948: sens_default:       0     1949: flags:           0x00
1984: device_select:      0
1984: vdd_rail:    255
1984: vddio_rail:    255
drv_cfg[3]
-----------------------------
item-registry-ID: name: value...
1950-1951: UUID: (UNKNOWN DRV) 24da648d-5449-4753-bc8a-3e2cc9f4e4b0
1952: off_to_idle:     5700     1953: idle_to_ready:  10000     1954: i2c_bus:         0x04
1955: reg_group_id:    1040     1956: cal_grp_id:     65535     1957: gpio1:         0x002b
1958: gpio2:         0xffff     1959: sensor_id:         40     1960: i2c_address:     0x48
1961: data_type1:         5     1962: data_type2:         6     1963: rel_sns_idx:       -1
1964: sens_default:       0     1965: flags:           0x80
1985: device_select:      0
1985: vdd_rail:    147
1985: vddio_rail:      2
drv_cfg[4]
-----------------------------
item-registry-ID: name: value...
1966-1967: UUID: NULL 00000000-0000-0000-0000-000000000000
1968: off_to_idle:        0     1969: idle_to_ready:      0     1970: i2c_bus:         0x00
1971: reg_group_id:       0     1972: cal_grp_id:         0     1973: gpio1:         0x0000
1974: gpio2:         0x0000     1975: sensor_id:          0     1976: i2c_address:     0x00
1977: data_type1:         0     1978: data_type2:         0     1979: rel_sns_idx:        0
1980: sens_default:       0     1981: flags:           0x00
1986: device_select:      0
1986: vdd_rail:    255
1986: vddio_rail:    255
SAM Registry data:
SSI Group registry read successful
```

從上述command可以知道我在sensor_def_qcomdev.conf並沒有更新到，所以合理懷疑有一個地方是存放sensor資料並且使用qfil也無法抹除與更新

下面這個command可以確認那個sensor有被成功初始化，目前看起來只有G-Sensor被初始化成功

```
console:/ # sns_dsps_tc0001
Starting sns_dsps_tc0001
Retrieved all sensor info
Sensor Name LIS3DH Accelerometer
Vendor Name STMicroelectronics
Sensor id 0     DataType 0
Version 1       MaxSampleRate 200       IdlePower 1
Max Power 11    Max Range 10283018      Resolution 643
```

其實android在第一次開機時會從*/vendor/etc/sensors/sensor_def_qcomdev.conf*製作出sns.reg檔

剛剛上面的command其實就是從*/mnt/vendor/persist/sensors/sns.reg*內去撈資料

要把sensor_def_qcomdev.conf重新更新sns.reg有兩種作法

第一種方法就是把sns.reg刪掉，但這必須android在root狀態下才可弄

```
console:/ # su
console:/ # mount -o rw,remount /mnt
console:/ # rm /mnt/vendor/persist/sensors/sns.reg
console:/ # reboot
```

重新開機後，就會根據sensor_def_qcomdev.conf產生一份新的sns.reg

第二種方法就是*/vendor/etc/sensors/sensor_def_qcomdev.conf*需要去做進版的動作

Ver 1.1

```
:version 0x00010001
```

改成Ver 1.2

```
:version 0x00010002
```

