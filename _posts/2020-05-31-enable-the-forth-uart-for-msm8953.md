---
layout: post
title: "Enable The Forth Uart for MSM8953"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle:
visualworkflow: true
---

MSM8953按照規格來說總共有四組Uart可以使用，只是第四組Uart的pin預設是被當成SPI，所以需要在Device Tree中關掉SPI並啟用Uart才行

![image-20200531012353287]({{site.baseurl}}/img/image-20200531012353287.png)

從Qualcomm那邊的DTS中可以獲得一些有用的資訊
*kernel/msm-4.9/arch/arm64/boot/dts/qcom/msm8953.dtsi*

	blsp1_uart0: serial@78af000 {
			compatible = "qcom,msm-uartdm-v1.4", "qcom,msm-uartdm";
			reg = <0x78af000 0x200>;
			interrupts = <0 107 0>;
			clocks = <&clock_gcc clk_gcc_blsp1_uart1_apps_clk>,
				<&clock_gcc clk_gcc_blsp1_ahb_clk>;
			clock-names = "core", "iface";
			pinctrl-names = "default";
			pinctrl-0 = <&uart_console_active>;
			status = "ok";
		};
	
	blsp2_uart0_ls: serial@7aef000 {
	    compatible = "qcom,msm-uartdm-v1.4", "qcom,msm-uartdm";
	    reg = <0x7aef000 0x200>;
	    interrupts = <0 306 0>;
	    clocks = <&clock_gcc clk_gcc_blsp2_uart1_apps_clk>,
	  	  <&clock_gcc clk_gcc_blsp2_ahb_clk>;
	    clock-names = "core", "iface";
		pinctrl-names = "default","sleep";
		pinctrl-0 = <&hsuart_active>;
		pinctrl-1 = <&hsuart_sleep>;
		status = "ok";
	};
	
	blsp2_uart0_gs: uart@7aef000 {
		compatible = "qcom,msm-hsuart-v14";
		reg = <0x7aef000 0x200>,
			<0x7ac4000 0x1f000>;
		reg-names = "core_mem", "bam_mem";
	
		interrupt-names = "core_irq", "bam_irq", "wakeup_irq";
		#address-cells = <0>;
		interrupt-parent = <&blsp2_uart0_gs>;
		interrupts = <0 1 2>;
		#interrupt-cells = <1>;
		interrupt-map-mask = <0xffffffff>;
		interrupt-map = <0 &intc 0 306 0
				1 &intc 0 239 0
				2 &tlmm 17 0>;
	
		qcom,inject-rx-on-wakeup;
		qcom,rx-char-to-inject = <0xFD>;
		qcom,master-id = <84>;
		clock-names = "core_clk", "iface_clk";
		clocks = <&clock_gcc clk_gcc_blsp2_uart1_apps_clk>,
			<&clock_gcc clk_gcc_blsp2_ahb_clk>;
		pinctrl-names = "sleep", "default";
		pinctrl-0 = <&hsuart_sleep>;
		pinctrl-1 = <&hsuart_active>;
		qcom,bam-tx-ep-pipe-index = <0>;
		qcom,bam-rx-ep-pipe-index = <1>;
		qcom,msm-bus,name = "blsp2_uart0_gs";
		qcom,msm-bus,num-cases = <2>;
		qcom,msm-bus,num-paths = <1>;
		qcom,msm-bus,vectors-KBps =
				<84 512 0 0>,
				<84 512 500 800>;
		status = "disabled";
	};
	
	blsp2_uart1: uart@7af0000 {
		compatible = "qcom,msm-hsuart-v14";
		reg = <0x7af0000 0x200>,
			<0x7ac4000 0x1f000>;
		reg-names = "core_mem", "bam_mem";
	
		interrupt-names = "core_irq", "bam_irq", "wakeup_irq";
		#address-cells = <0>;
		interrupt-parent = <&blsp2_uart1>;
		interrupts = <0 1 2>;
		#interrupt-cells = <1>;
		interrupt-map-mask = <0xffffffff>;
		interrupt-map = <0 &intc 0 307 0
				1 &intc 0 239 0
				2 &tlmm 21 0>;
	
		qcom,inject-rx-on-wakeup;
		qcom,rx-char-to-inject = <0xFD>;
		qcom,master-id = <84>;
		clock-names = "core_clk", "iface_clk";
		clocks = <&clock_gcc clk_gcc_blsp2_uart2_apps_clk>,
			<&clock_gcc clk_gcc_blsp2_ahb_clk>;
		pinctrl-names = "sleep", "default";
		pinctrl-0 = <&blsp2_uart1_sleep>;
		pinctrl-1 = <&blsp2_uart1_active>;
		qcom,bam-tx-ep-pipe-index = <2>;
		qcom,bam-rx-ep-pipe-index = <3>;
		qcom,msm-bus,name = "blsp2_uart1";
		qcom,msm-bus,num-cases = <2>;
		qcom,msm-bus,num-paths = <1>;
		qcom,msm-bus,vectors-KBps =
				<84 512 0 0>,
				<84 512 500 800>;
		status = "disabled";
	};
	
	blsp1_serial1: serial@78b0000 {
		compatible = "qcom,msm-uartdm-v1.4", "qcom,msm-uartdm";
		reg = <0x78b0000 0x200>;
		interrupts = <0 108 0>;
		clocks = <&clock_gcc clk_gcc_blsp1_uart2_apps_clk>,
			<&clock_gcc clk_gcc_blsp1_ahb_clk>;
		clock-names = "core", "iface";
		pinctrl-names = "default","sleep";
		pinctrl-0 = <&uart4_active>;
		pinctrl-1 = <&uart4_sleep>;
		status = "ok";
	};
由以上DTS可以得知，預設已經啟用三組Uart

blsp1_uart0 (serial@78af000) -> qcom,msm-uartdm( Low Speed Uart)
blsp1_serial1(serial@78b0000) -> qcom,msm-uartdm( Low Speed Uart)

blsp2_uart0_ls (serial@7aef000) -> qcom,msm-uartdm( Low Speed Uart)

而第四組uart如下是被設成不啟用，這邊要設為啟用，就是把status="ok"
blsp2_uart1 (uart@7af0000)

另外就是要注意的是，這邊Driver並不是使用Low speed uart(qcom,msm-uartdm )而是"qcom,msm-hsuart-v14"，所以系統啟動時生成的裝置檔被命名為ttyHSx，如果是Low Speed Uart則是ttyHSLx

也可固定ttyHSx要指定那個號碼，例如，指定blsp2_uart1 (uart@7af0000)連結到ttyHS1

	aliases {
		......
		uart1 = &blsp2_uart1; 
	};
如果是指定ttyHSLx則是
serial0 = &blsp1_uart0;  	<- ttyHSL0
serial1 = &blsp2_uart0_ls; 	<-ttyHSL1

再來就是如果嘗試把第四組uart的 DTS改成Low Speed Uart會發現裝置檔開機還是有出現三個
ttyHSL0、ttyHSL1、ttyHSL2
看demsg會發現第四組Uart（7af0000）的確是有吃到並成為ttyHSL2
但原本uart(78b0000)就消失不見了

```
[    4.085717] msm_serial 78af000.serial: msm_serial: detected port #0
[    4.086969] msm_serial 78af000.serial: uartclk = 7372800
[    4.098581] msm_serial: console setup on port #0
[    4.109724] bootconsole [msm_serial_dm0] disabled
[    4.118960] msm_serial 7aef000.serial: msm_serial: detected port #1
[    4.122798] msm_serial 7aef000.serial: uartclk = 19200000
[    4.135263] msm_serial 7af0000.serial: msm_serial: detected port #2
[    4.143038] msm_serial 7af0000.serial: uartclk = 19200000
[    4.156271] msm_serial: driver initialized

```

看msm_serial.c會發現Low speed uart在probe時會scan uart dts由上而下，並且限制最多只能吃三個
為何會這樣限定尚不知道，目前啟用第四組只能當high speed uart