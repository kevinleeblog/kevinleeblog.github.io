---
layout: post
title: "Arduino開發使用Eclipse"
author:     Kevin Lee
tags: 		Arduino Eclipse
subtitle:   
category:  project1
visualworkflow: true
---
# 前言

使用官方提供的IDE有一個缺點，就是沒有程式碼自動補齊的功能，變成每一行程式碼每一個字都要一個一個手動去key in。我個人是比較喜好開源的IDE，像是Eclipse等。

Eclipse可以透過Plugin的方式來開發Arduino

底下就是說明如何去安裝這個Plugin



# 安裝

安裝Arduino Plugin的方式可以透過Eclipse Marketplace來安裝



![image-20181204094407654](../img/image-20181204094407654-3887847.png)

在Find中輸入關鍵字**arduino**
這裡要安裝兩個plugin，**The Arduino Eclipse plugin named Sloeber V4**和**Eclipse C++ IDE For Arduino 3.0**

![image-20181204094631649](../img/image-20181204094631649-3887991.png)



安裝完後會自動重啟Eclipse，此時就可以開啟Arduino專案了



# 使用

開啟新專案

![image-20181204095614797](../img/image-20181204095614797-3888574.png)



輸入Project Name後，按下Next

![image-20181204095713158](../img/image-20181204095713158-3888633.png)



設定使用的Board和Port後按下next

![image-20181204095826541](../img/image-20181204095826541-3888706.png)



這邊就用default的按下Finish完成

![image-20181204095936348](../img/image-20181204095936348-3888776.png)



把Demo.ino檔案打開

先設定Uart，使用自動補齊的方式
輸入**Se**後，在mac上快捷鍵是**「option + \」**，會出現如下的小視窗，選擇Serial

![image-20181204102726552](../img/image-20181204102726552-3890446.png)

![image-20181204103121428](../img/image-20181204103121428-3890681.png)



# 編譯與上傳

透過上方工具列執行編譯並上傳的動作

![image-20181204103724744](../img/image-20181204103724744-3891044.png)

顯示Uart Console

![image-20181204103902058](../img/image-20181204103902058-3891142.png)

