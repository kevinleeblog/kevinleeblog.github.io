---
layout: post
title: "查詢GPIO狀況"
author:     Kevin Lee
tags: 		Yocto-Study 
subtitle:   
category:  project1
visualworkflow: true
---
有時候會想要知道這個Platform的GPIO全部的狀態，包含是否被Kernel佔用等等

可以到/sys/kernel/debug/pinctrl/1e6e2000.syscon:pinctrl-aspeed-g5-pinctrl來找資訊

```
# cat pingroups |less
registered pin groups:
group: ACPI
pin 192 (R22)
pin 193 (R21)
pin 194 (P22)
pin 195 (P21)
pin 200 (Y20)
pin 201 (AB20)
pin 202 (AB21)
pin 203 (AA21)
...
```

想要知道是否哪些gpio pin已經被佔用

```
# cat pinmux-pins
Pinmux settings per pin
Format: pin (name): mux_owner|gpio_owner (strict) hog?
pin 0 (B14): UNCLAIMED
pin 1 (D14): UNCLAIMED
pin 2 (D13): UNCLAIMED
pin 3 (E13): UNCLAIMED
pin 4 (C14): UNCLAIMED
pin 5 (A13): UNCLAIMED
pin 6 (C13): device 1e680000.ethernet function MDIO2 group MDIO2
pin 7 (B13): device 1e680000.ethernet function MDIO2 group MDIO2
pin 8 (K19): UNCLAIMED
pin 9 (L19): UNCLAIMED
pin 10 (L18): UNCLAIMED
pin 11 (K18): UNCLAIMED
pin 12 (J20): UNCLAIMED
pin 13 (H21): UNCLAIMED
```

可以知道pin6和pin7已經被Kernel給佔用了