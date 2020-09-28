---
layout: post
title: "Enable Uart 6 for MSM8953 with low speed driver"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle:
visualworkflow: true
---

Quclcomm MSM8953共有4組Uart，其中有一個已經被設為Debug用
可用的實際上只剩下三個
而MSM8953的Serial Driver有分高速與低速版本，之前Bring up時有發現低速版本只能列舉到三組uart，所以最後一組uart也就是uart6我讓他使用高速版本的Driver

但後來有使用到此高速版本uart的同事反應，當他們的應用如果週邊裝置處理的時間過長，他們在uart RX就會收到不對的字元0xFD，但是如果週邊裝置處理的時間是即時的，那uart的接收就正常

聽他們的描述之後，腦海中閃過一個畫面0xFD

*msm8953.dtsi*

```dtd
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

                qcom,tx-gpio = <&tlmm 20 0x00>;
                qcom,rx-gpio = <&tlmm 21 0x00>;
                qcom,cts-gpio = <&tlmm 22 0x00>;
                qcom,rfr-gpio = <&tlmm 23 0x00>;

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
```

0xFD在上述的Device Tree中有出現，但這是表示什麼呢？

*devicetree/bindings/serial/msm_serial_hs.txt*

- **qcom, wakeup_irq** : UART RX GPIO IRQ line to be configured as wakeup source.
- **qcom,inject-rx-on-wakeup** : inject_rx_on_wakeup enables feature where on
receiving interrupt with UART RX GPIO IRQ line (i.e. above wakeup_irq property),
HSUART driver injects provided character with property rx_to_inject.
- **qcom, rx-char-to-inject** : The character to be inserted on wakeup.

應該可以理解的就是這組高速uart driver當平常沒使用時，就會進入到休眠狀態而要喚醒它，Driver會先傳一個0xFD字元來做喚醒字元，有點像是Wake-on-Lan一樣傳送一個Magic Packet來喚醒電腦一樣

所以如果要改成合乎我們的應用可能有三種方式

- 請他們改程式
- 我修改高速版本的Driver讓他們可以使用
- 捨棄高速版本Driver，改用低速版本

後來我選擇改用低速版本的Driver，這邊需要修改一些程式

Uart6增加低速版本的Device Tree
*msm8953.dtsi*

```
	/* uart6 */
	blsp2_uart1_ls: serial@7af0000 {
        compatible = "qcom,msm-uartdm-v1.4", "qcom,msm-uartdm";
        reg = <0x7af0000 0x200>;
        interrupts = <0 307 0>;
        clock-names = "core", "iface";
        clocks = <&clock_gcc clk_gcc_blsp2_uart2_apps_clk>,
                <&clock_gcc clk_gcc_blsp2_ahb_clk>;
        pinctrl-names = "default";
        pinctrl-0 = <&uart6_active>;
        status = "ok";
	};

```

*msm8953-pinctrl.dtsi*

```
		    uart6_active: uart6_active {
				mux {
				        pins = "gpio20", "gpio21";
				        function = "blsp_uart6";
				};

				config {
				        pins = "gpio20", "gpio21";
				        drive-strength = <8>;
				        bias-disable;
				};
    		};
```

修改低速版本Driver，讓Uart 6使用到低速

*kernel/msm-4.9/drivers/tty/serial/msm_serial.c*

```xml-dtd
diff --git a/kernel/msm-4.9/drivers/tty/serial/msm_serial.c b/kernel/msm-4.9/drivers/tty/serial/msm_serial.c
index 3367d9f..1c0a78a 100644
--- a/kernel/msm-4.9/drivers/tty/serial/msm_serial.c
+++ b/kernel/msm-4.9/drivers/tty/serial/msm_serial.c
@@ -1590,6 +1590,16 @@ static struct msm_port msm_uart_ports[] = {
                        .line = 2,
                },
        },
+       {
+               .uart = {
+                       .iotype = UPIO_MEM,
+                       .ops = &msm_uart_pops,
+                       .flags = UPF_BOOT_AUTOCONF,
+                       .fifosize = 64,
+                       .line = 3,
+
+               },
+       },
 };

```

如果Uart 6有成功吃進去，就會在系統看到/dev/ttyHSL3