---
layout: post
title: "ILITEK Touch Panel Porting"
auther: Kevin Lee
category: project1
tags: [Android_ROM_Design, Job_Logging]
subtitle:
visualworkflow: true
---

### Touch Panel Interfaces

The following figure shows a reference design for touch panel interfaces.

![hz9TSRI]({{site.baseurl}}/img/hz9TSRI.png)

![eUpwEos]({{site.baseurl}}/img/eUpwEos.png)

### Porting ILITEK TouchScreen driver and DTS

#### DTS

For Quectel SC60 

Hardware reference
![424rJ2N]({{site.baseurl}}/img/424rJ2N.png)

![zlwn7tX]({{site.baseurl}}/img/zlwn7tX.png)

According to SC60 pin defined,
TP0_I2C_SCL and TP0_I2C_SDK connnect to I2C3
![akNm9QC]({{site.baseurl}}/img/akNm9QC.png)

TP0_RST connects to GPIO_64
TP0_INT connects to GPIO_65

So, added the its content
kernel/msm-3.18/arch/arm/boot/dts/qcom/msm8953-mtp.dtsi

```
&i2c_3 {  /* BLSP1 QUP3 */ 
    status = "ok";
        ilitek@41 {
            compatible = "tchip,ilitek";
            reg = <0x41>;
            interrupt-parent = <&tlmm>;
            interrupts = <65 0x2>;
            vdd-supply = <&pm8953_l10>;
            vcc_i2c-supply = <&pm8953_l6>;
            ilitek,reset-gpio = <&tlmm 64 0x0>;
            ilitek,irq-gpio = <&tlmm 65 0x2008>;
            ilitek,vbus = "vcc_i2c";
            ilitek,vdd = "vdd";
            ilitek,name = "ilitek_i2c";
            pinctrl-names = "pmx_ts_active","pmx_ts_suspend";
            pinctrl-0 = <&ts_int_active &ts_reset_active>;
            pinctrl-1 = <&ts_int_suspend &ts_reset_suspend>;
    };
};
```

```
Required properties:
- compatible           : should be "focaltech,5x06".
- reg                  : i2c slave address of the device.
- interrupt-parent     : parent of interrupt.
- interrupts           : touch sample interrupt to indicate presense or release of fingers on the panel.
- vdd-supply           : Power supply needed to power up the device.
- vcc_i2c-supply       : Power source required to power up i2c bus.
- ilitek,irq-gpio   : irq gpio which is to provide interrupts to host, same as "interrupts" node. It will also contain active low or active high information.
- ilitek,reset-gpio : reset gpio to control the reset of chip.
- ilitek,name       : name of the controller
- pinctrl-names : This should be defined if a target uses pinctrl framework.
                        See "pinctrl" in Documentation/devicetree/bindings/pinctrl/msm-pinctrl.txt.
                        Specify the names of the configs that pinctrl can install in driver.
                        Following are the pinctrl configs that can be installed:
                        "pmx_ts_active" : Active configuration of pins, this should specify active
                        config defined in pin groups of interrupt and reset gpio.
                        "pmx_ts_suspend" : Disabled configuration of pins, this should specify sleep
                        config defined in pin groups of interrupt and reset gpio.
                        "pmx_ts_release" : Release configuration of pins, this should specify
                        release config defined in pin groups of interrupt and reset gpio.
```

#### Download ILITEK driver

```
$ git clone https://github.com/NovasomIndustries/ilitek_module_1.0.0.git

$ cd ilitek_module_1.0.0/

$ mkdir kernel/msm-3.18/drivers/input/touchscreen/ilitek_module

$ git archive master | tar -x -C kernel/msm-3.18/drivers/input/touchscreen/ilitek_module/
```

#### Modify ilitek Makefile

```
$ vim kernel/msm-3.18/drivers/input/touchscreen/ilitek_module/Makefile
obj-y += ilitek_main.o \
ilitek_platform_init.o \
ilitek_update.o \
ilitek_tool.o
```

kernel/msm-3.18/drivers/input/touchscreen/Makefile
![MSv4Acb]({{site.baseurl}}/img/MSv4Acb.png)

kernel/msm-3.18/drivers/input/touchscreen/Kconfig
![PRrAtNZ]({{site.baseurl}}/img/PRrAtNZ.png)

#### Add driver to Kernel Config

```
$ vim kernel/msm-3.18/arch/arm64/configs/msmcortex-perf_defconfig
CONFIG_TOUCHSCREEN_ILITEK_DRIVER=y
$ vim kernel/msm-3.18/arch/arm64/configs/msmcortex_defconfig
CONFIG_TOUCHSCREEN_ILITEK_DRIVER=y
```