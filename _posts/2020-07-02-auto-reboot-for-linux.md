---
layout: post
title: "Linux的自動開機程式"
auther: Kevin Lee
category: project1
tags: [Ubuntu]
subtitle: Ubuntu 18.04 Lts
visualworkflow: true
---

這是BQE單位提出希望我能在Linux環境下寫個自動開關機程式，這個程式的特徵很簡單就是開機進去等30秒後在自動重新開機，總共要跑2000次

剛聽聞此需求腦中浮現幾種可以嘗試的作法，但我喜歡自己幻想在看自己的幻想能不能實現，我想用的方式就是Linux Shell Script結合Crontab

Crontab是Linux預設啟動與提供的例行性工作服務，會想使用是因為我不想在多寫Systemd去執行Auto Reboot Shell Script，我的原則就是系統有的就用系統的，沒有或不足的才自己DIY。但Crontab還是有限制就是最小執行的時間單位不是秒而是分，造BQE的要求要30秒顯然會多一點時間，至少會多2000*30=60000，大約會多16小時，反正就先試試看可不可以過關，不行我也還有B計畫

腦中幻想就是crontab固定一分鐘去執行一個名為autoRebootForCron.sh

```
＄ sudo crontab -e
*/1 * * * * bash /boot/autoRebootForCron.sh
```

會加上sudo是因為我發現reboot似乎只有root權限才能執行

autoRebootForCron.sh

```
$ sudo gedit /boot/autoRebootForCron.sh
#!/bin/bash

if [ -z $1]
then
	declare -i time=2000
else
	declare -i time=$1
fi

## Main Function
[ -e /boot/enableRebootTime ] && {
	echo $time > /boot/TotalRebootTimes
	declare -i enableRebootTime=$(cat /boot/enableRebootTime)
	echo "$[ $enableRebootTime+1 ]" > /boot/enableRebootTime
	echo $enableRebootTime > /boot/CurrentRebootTimes
	/sbin/reboot
	if [ $enableRebootTime -eq $time ]
	then
		rm /boot/enableRebootTime
	fi
	
}


$ sudo chmod a+x /boot/autoRebootForCron.sh
```

autoRebootForCron.sh要找個地方放，我都放在/boot下

如何使用

剛剛crontab已經設好了，例行性工作也固定一分鐘會去執行此script，但此script要啟用還必須加上開關才會開始動作

```
$ sudo touch /boot/enableRebootTime
```

到此就開始自動Reboot到2000次了

/boot下還有一些資訊可看

/boot/CurrentRebootTimes表示目前已經reboot的次數
/boot/TotalRebootTimes表示總共預定要執行的reboot次數
/boot/enableRebootTime表示執行或關閉reboot的開關
