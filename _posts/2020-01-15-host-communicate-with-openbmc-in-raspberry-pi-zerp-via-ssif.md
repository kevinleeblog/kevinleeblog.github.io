---
layout: post
title: "Host OS communicate to raspberry pi with IPMI via SSIF "
auther: Kevin Lee
category: project1
tags: [OpenBMC,Raspberry_Pi]
subtitle:
visualworkflow: true
---

### 為何做？

先前我已經成功把OpenBMC安裝於Raspberry Pi Zero W上，也學習了OpenBMC如何透過DBus傳送IPMI Messaging，為了要更深入了解BMC，就是要實際架設伺服器，這邊我和公司借了一塊伺服器主機板，上面使用的Chipset為伺服器等級的Intel C246

![image-20200115105440391]({{site.baseurl}}/img/image-20200115105440391.png)

第一件想做的事情就是讓Host OS可以與BMC(Paspberry Pi)直接溝通，因為沒有LPC，所以我想透過SSIF變成BMC與Host Host溝通的橋樑，這段網路上的資料比較少，所以我也要努力Study IPMI關於SSIF這段與C246的spec

目前OpenBMC尚未實現SSIF這段，所以這段要由我DIY去實現

### 如何做？

SSIF主要是透過PCH SMBus Host Controller並且BMC本身是當Slave角色，而我使用的Raspberry Pi Zero W其MCU為BCM2835，根據spec，BCM2835有專門提供BSCSL當I2C/SPI Slave使用
![image-20200117075339879]({{site.baseurl}}/img/image-20200117075339879.png)

![image-20200117081557535]({{site.baseurl}}/img/image-20200117081557535.png)

**由spec可以看出BSCSL使用GPIO18~21共4pin，模式是ALT3**

![image-20200117081714203]({{site.baseurl}}/img/image-20200117081714203.png)

**Raspberry Pi Zero w的接腳**

![image-20200117082528638]({{site.baseurl}}/img/image-20200117082528638.png)

但是目前DTS沒有設定到這個register，所以需要自己寫
由spec可以獲得一些資訊

* GPIO18：BSCSL_SDA         GPIO19：BSCSL_SCL
* BSCSL Base Register address:0x7E21_4000

創建linux-raspberrypi/arch/arm/boot/dts/overlays/i2cslave-bcm2708-overlay.dts

```dtd
/*
 * Device tree overlay for i2c_bcm2708, i2cslave bus
 *
 * Compile:
 * dtc -@ -I dts -O dtb -o i2cslave-bcm2708-overlay.dtbo i2cslave-bcm2708-overlay.dts
 */
#include <dt-bindings/clock/bcm2835.h>
#include <dt-bindings/pinctrl/bcm2835.h>
/dts-v1/;
/plugin/;

/{
	compatible = "brcm,bcm2707", "brcm,bcm2708", "brcm,bcm2709", "brcm,bcm2835";

	fragment@0{
		target = <&soc>;
		__overlay__ {
			i2cslv0: i2c@7e214000 {
			compatible = "brcm,bcm2835-i2c-slave";
			reg = <0x7e214000 0x1000>;
			interrupts = <2 11>;
			/*clocks = <&clk_core>;*/
			clocks = <&clocks BCM2835_CLOCK_VPU>;
			clock-frequency = <100000>;
			#address-cells = <1>;
			#size-cells = <1>;
			pinctrl-names = "default";
      pinctrl-0 = <&i2cslv0_pins>;
			status = "okay";
			};
		};
	};
	
	fragment@1{
		target = <&gpio>;
		__overlay__{
			i2cslv0_pins: i2cslv0 {
				brcm,pins = <18 19>;
        brcm,function = <BCM2835_FSEL_ALT3>; /* alt3 */	
			};
		};
	};
	
	fragment@2{
		target-path = "/aliases";
		__overlay__{
			i2cslv0 = "/soc/i2c@7e214000";
		};
	};
	
	fragment@3{
		target-path = "/__symbols__";
		__overlay__{
			i2cslv0 = "/soc/i2c@7e214000";
			i2cslv0_pins = "/soc/gpio@7e200000/i2cslv0";
		};
	};
};
```

載入Raspberry Pi Zero W的i2c-dev module

```
$ modprobe i2c-dev
```

但是每次重開機就要重設一次
永久生效

```
$ vi /etc/modules-load.d/i2c-dev.conf
i2c-dev
```

重新開機就好

我這邊用Raspberry Pi Zero W，只有看到一個i2c設備

```
root@raspberrypi0-wifi:~# ls /dev/i2c*  
/dev/i2c-2
```

