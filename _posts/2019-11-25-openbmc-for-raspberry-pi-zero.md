---
layout: post
title: "Raspberry Pi zero Bring up OpenBMC"
auther: Kevin Lee
category: project1
tags: [OpenBMC]
subtitle:
visualworkflow: true
---

### Why to do?

I don't have any server machines actually. So, I need a real platform to learn IPMI and openbmc. I knew that OpenBMC can run on raspberry pi, but it seems that there are some bugs which need to be solved. That's why I wrote this to note how I bring up OpenBMC for Raspberry Pi.

### How to do?

When I download openbmc source and start to build, first problem is `Image is too large`
I had solved this issue in [Compile OpenBMC For Raspberry Pi](/2019/11/11/openbmc-install-raspberry-pi)

And next, It need to flash to SD card. But, it didn't have suitable image format for Raspberry Pi if you used config file from OpenBMC.
So, I need to add the custom image format for Raspberry Pi in `local.conf` and some config parameter.

My platform is *Raspberry Pi Zero W*, so modify *MACHINE ??= "raspberrypi0-wifi"*

![image-20191125113713081]({{site.baseurl}}/img/image-20191125113713081.png)

Modify local.conf to Install ipmitool and wpa-supplicant

```
IMAGE_INSTALL_append = " ipmitool wpa-supplicant"
```

Because there is not ethernet interface on Raspberry Pi Zero W, I need to *wpa-supplicant* to set wifi. 

#### Start To Flash

Go into *tmp/deploy/images/raspberrypi0-wifi/* and prepare SD card.

```
$ cd tmp/deploy/images/raspberrypi0-wifi/
$ lsblk
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdd                     8:48   1   7.4G  0 disk 
├─sdd2                  8:50   1   164M  0 part 
└─sdd1                  8:49   1    40M  0 part 
sdb                     8:16   0 953.9G  0 disk 
├─sdb2                  8:18   0     1K  0 part 
├─sdb5                  8:21   0 953.2G  0 part 
│ ├─ubuntu--vg-swap_1 253:1    0   976M  0 lvm  
│ └─ubuntu--vg-root   253:0    0 930.4G  0 lvm  /
└─sdb1                  8:17   0   731M  0 part /boot
sdc                     8:32   0 931.5G  0 disk 
└─sdc1                  8:33   0 931.5G  0 part /mnt/d1d466d5-3139-4049-b4a1-b5c0e1384111
sda                     8:0    0 238.5G  0 disk 
└─sda1                  8:1    0 238.5G  0 part /media/kevinlee/9949c5c6-6e94-4631-824e-61b9c20b5742
$ sudo umount /dev/sdd*
$ sudo dd if=obmc-phosphor-image-raspberrypi0-wifi-20191125050352.rootfs.rpi-sdimg of=/dev/sdd
$ sync
```

Power on Raspberry Pi Zero

![20191125_131550]({{site.baseurl}}/img/20191125_131550.jpg)

![20191125_131559]({{site.baseurl}}/img/20191125_131559.jpg)

Setting my wifi on Raspberry Pi

Modify */etc/wpa_supplicant.conf*. My Wifi router is hidden.

```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=0
update_config=1

network={
				ssid="yourHiddenSSID"
        scan_ssid=1
        psk=Your_wifi_password
}
```

Connect to internet

```
$ wpa_supplicant -iwlan0 -c /etc/wpa_supplicant.conf
```

