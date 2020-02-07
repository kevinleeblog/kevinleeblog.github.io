---
layout: post
title: "Openbmc Studying:ipmitool"
auther: Kevin Lee
category: project1
tags: [OpenBMC]
subtitle:
visualworkflow: true
---

如果在BMC內下
`$ ipmitool raw 06 01`

好奇到底程式流程會怎麼跑？

從下圖來看，*ipmi_run_main*內會把`06 01`拆成netfn, lun和cmd並做成ipmi request message
最後ipmi_dbus_sendrecv經由DBus負責傳遞ipmi request message，然後等待DBus回傳ipmi response message，回傳的

* Completion Code
* data len用來表示要回傳的data長度
* data

![image-20200108110931628]({{site.baseurl}}/img/image-20200108110931628.png)



channel=0x0表示是使用IPMB Protocol和BMC溝通

```
root@raspberrypi0-wifi:~# ipmitool raw 06 01 -vvv
Running Get PICMG Properties my_addr 0x20, transit 0, target 0
Error response 0xc1 from Get PICMG Properties
Running Get VSO Capabilities my_addr 0x20, transit 0, target 0
Invalid completion code received: Invalid command
Acquire IPMB address
Discovered IPMB address 0x0
Interface address: my_addr 0x20 transit 0:0 target 0x20:0 ipmb_target 0

RAW REQ (channel=0x0 netfn=0x6 lun=0x0 cmd=0x1 data_len=0)
RAW RSP (15 bytes)
 00 00 00 00 02 00 00 00 00 00 00 00 00 00 00
```



----

看完ipmitool發送ipmi request message，那在OpenBMC內究竟是誰去接收這個request message?
由DBus Object來看是屬於xyz.openbmc_project.Ipmi.Host，所以創造這個Object的Package也就是會接收的package。

Phosphor-ipmi-host就是這個package