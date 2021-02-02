---
layout: post
title: "Android open source project (AOSP) in Docker"
auther: Kevin Lee
category: project1
tags: [Docker]
subtitle:
visualworkflow: true
---

雖然開發aosp最適合是在Linux環境上開發，但有些原廠提供的工具卻是只能在windows環境上執行，這個時候就有兩種作法了
一)是使用兩台電腦，一台是Linxu，一台是Windows，透過網芳等很多方式來傳檔案
二)是使用Virtual Machine，只要一台裝有windows環境的電腦就好，在上面安裝vmware或virtualbox之類的軟體，並建立一個虛擬Linux環境上去編譯Aosp專案

經過許多年，大部分的同事包含我自己都是用以上方式來建立我的專案，直到...Containerization容器化出現

會想把aosp編譯環境換到Docker上，主要的點就是效能比VM好太多了，Container Engine直接和windows底層溝通資源，比起VM的Hypervisor Hosted type來的有效率多了

廢話不多說，直接看Dockerfile，Docker環境的安裝建置不再我這篇的主題重點
首先開啟文字編輯器，並copy以下內容

```
FROM ubuntu:18.04

MAINTAINER kevinlee

RUN apt-get update -y \
## My favorite tools
&& apt-get install ack vim screen geany tree locate \
## ssh server
openssh-server sudo \
## Android BSP compiler packages
git-core gnupg flex bison build-essential \
zip curl zlib1g-dev gcc-multilib \
g++-multilib libc6-dev-i386 \
lib32ncurses5-dev x11proto-core-dev kmod \
libx11-dev lib32z1-dev libgl1-mesa-dev openjdk-8-jdk python mkisofs libssl-dev bc rsync \
libxml2-utils xsltproc unzip fontconfig liblz4-tool libxkbfile1 libxss1 \
-y

RUN useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 kevinlee
RUN  echo 'kevinlee:kevinlee' | chpasswd
RUN service ssh start

WORKDIR /home/ubuntu
#COPY file .
CMD ["declare","-x","DISPLAY=:0"]
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

檔案存為「**Dockerfile**」，一定要這個檔案名稱

這個Dockerfile是我在編譯aosp時有缺少什麼就補上什麼套件，使用ubuntu 18.04環境，我沒有安裝桌面環境，想要顯示Gui我是用ssh搭配Mobaxterm來達到這件事，IDE是使用vscode，有擴充套件可以顯示container內部的aosp程式碼，帳號密碼都是kevinlee
對我開發專案而言以上就夠用了，不夠用之後再加

使用Docker Engine編譯Dockerfile並製成image

```
$ docker build -t aosp-project .
```

跑大約要10分鐘以上，看環境硬體網路效能

我們需要一個固定存放 aosp程式碼的空間，即使container被移除裡面的資料還是存在

```
$ docker volume create android-pro
```

使用Docker engine實例化aosp-project image

```
$ docker run --name my-aosp-pro --privileged -p 5566:22 --mount type=bind,source="$(pwd)"/OUT,target=/OUT -v android-pro:/volume -it aosp-project
```

建立完成後就可以使用Mobaxterm透過ssh連到container內部

```
$ ssh -p 5566 kevinlee@localhost
```

![image-20210202094411871]({{site.baseurl}}/img/image-20210202094411871.png)

有一個「X11 forwarding request failed on channel 0」錯誤，表示我Dockerfile沒有寫好...
這會影響到不能使ssh gui不能顯示圖形，手動修改先，或有一天有空時再回來改Dockerfile

```
$ sudo vim /etc/ssh/sshd_config
```

![image-20210202095854779]({{site.baseurl}}/img/image-20210202095854779.png)

改以上三行內容後，重啟ssh

```
$ sudo /etc/init.d/ssh restart
```

根目錄下
OUT:可以和外部Windows電腦互傳檔案，當編譯好的aosp image就丟到此目錄給原廠工具燒錄
volume:就是存放aosp程式碼的地方

![image-20210202095203928]({{site.baseurl}}/img/image-20210202095203928.png)

![image-20210202100250772]({{site.baseurl}}/img/image-20210202100250772.png)