---
layout: post
title: "輕鬆建立Git Server於Windows環境"
auther: Kevin Lee
category: project1
tags: [Git]
subtitle:
visualworkflow: true
---

## 前言

有時候需要一些local端的Git Server先把專案上傳到此，等到開發一段時間後，在一次上傳到公司的Git Server，這就是為何我需要一個Local端的Git Server

而Gogs這套顯然可以滿足我的需求-在windows環境下架設免費的Git Server，雖然我的工作是開發android和Linux的，使用windows是因為原廠很多工具軟體還是需要在Windows下執行，而開發環境我也不用VM而是使用Docker來開發android bsp專案，在此應用下gogs可以符合我的需求。



## 下載安裝

請到[Gogs: A painless self-hosted Git service](https://gogs.io/)

點選**立即安裝**

![image-20210115095013048]({{site.baseurl}}/img/image-20210115095013048.png)

依照說明文件先安裝

1. Database
   我是默認使用SQLite3，就不需要額外安裝像是MySQL等資料庫了
2. Git
   [Git - Downloads (git-scm.com)](https://git-scm.com/downloads)
3. Gogs
   https://dl.gogs.io/0.12.3/gogs_0.12.3_windows_amd64.zip

解壓縮gogs，進入到解壓縮的目錄內執行./gogs web
gogs默認會在http://localhost:3000/ 啟動服務
第一次進入會需要做初始設定

然後就可以使用了，在開發專案時還可以紀錄一下開發過程

![image-20210115100605976]({{site.baseurl}}/img/image-20210115100612968.png)