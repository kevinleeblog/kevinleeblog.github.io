---
layout: post
title: "使app取得Root權限不用輸入密碼"
auther: Kevin Lee
category: project1
tags: [Ubuntu, Job_Logging]
subtitle: Ubuntu
visualworkflow: true
---

### 為何做？

昨天業務很急突然臨時找我來處理，一開始反應先是說客戶說執行app於Ubuntu會出現權限不足的錯誤。客戶給我看一下Source Code說是執行ioperm時這行時會需要輸入密碼，然後我看到有使用一些I/O Port，看來應是Ubuntu處理I/O Port時需要取得root權限，客戶反應要輸入帳密很不方便。

### 如何做？

我和客戶說Ubuntu有一種方式，可以指定某隻app使用sudo的取得root權限時，不用輸入密碼，但是執行程式時還是需要在執行app前加上sudo，但不用輸入密碼

比如說我希望以後執行apt去更新套件時都不需要輸入密碼，那我可以在最後面加上這行

```
$ sudo visudo
#includedir /etc/sudoers.d
kevinlee     ALL=(ALL) NOPASSWD: /usr/bin/apt
```

以後我取得root權限處理apt做套件安裝時都不用輸入密碼了

```
$ sudo apt update
已有:1 http://tw.archive.ubuntu.com/ubuntu xenial InRelease
已有:2 http://linux.teamviewer.com/deb stable InRelease                                                                                          
略過:3 http://dl.google.com/linux/chrome/deb stable InRelease                                                                                                              
下載:4 http://tw.archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]                                                                                               
已有:5 http://dl.google.com/linux/chrome/deb stable Release                                                                                                                                
下載:6 http://tw.archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]                                                                                                             
下載:8 http://tw.archive.ubuntu.com/ubuntu xenial-updates/main amd64 Packages [1,064 kB]                                                                                                   
已有:9 http://apt.insync.io/ubuntu xenial InRelease                                                                                                            
已有:10 http://ppa.launchpad.net/brightbox/ruby-ng/ubuntu xenial InRelease                                 
下載:11 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]                               
略過:12 http://us.archive.ubuntu.com/ubuntu trusty InRelease                                                   
下載:13 http://tw.archive.ubuntu.com/ubuntu xenial-updates/main i386 Packages [877 kB]                                    
已有:14 http://us.archive.ubuntu.com/ubuntu trusty Release                                                
下載:16 http://tw.archive.ubuntu.com/ubuntu xenial-updates/main Translation-en [410 kB]        
下載:17 http://tw.archive.ubuntu.com/ubuntu xenial-updates/main amd64 DEP-11 Metadata [322 kB]    
下載:18 http://tw.archive.ubuntu.com/ubuntu xenial-updates/main DEP-11 64x64 Icons [233 kB]         
下載:19 http://tw.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages [769 kB]           
下載:20 http://tw.archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages [698 kB]                       
下載:21 http://tw.archive.ubuntu.com/ubuntu xenial-updates/universe Translation-en [323 kB]                   
下載:22 http://tw.archive.ubuntu.com/ubuntu xenial-updates/universe amd64 DEP-11 Metadata [274 kB]                  
下載:23 http://tw.archive.ubuntu.com/ubuntu xenial-updates/universe DEP-11 64x64 Icons [411 kB]                       
下載:24 http://tw.archive.ubuntu.com/ubuntu xenial-updates/multiverse amd64 DEP-11 Metadata [5,964 B]
下載:25 http://tw.archive.ubuntu.com/ubuntu xenial-backports/main amd64 DEP-11 Metadata [3,324 B]
下載:26 http://tw.archive.ubuntu.com/ubuntu xenial-backports/universe amd64 DEP-11 Metadata [5,324 B]
下載:27 http://security.ubuntu.com/ubuntu xenial-security/main amd64 Packages [779 kB]      
下載:28 http://security.ubuntu.com/ubuntu xenial-security/main i386 Packages [612 kB]
```

