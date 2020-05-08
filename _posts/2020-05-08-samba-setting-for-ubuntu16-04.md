---
layout: post
title: "Samba Setting for Ubuntu 16.04"
auther: Kevin Lee
category: project1
tags: [Ubuntu]
subtitle: Ubuntu 16.04
visualworkflow: true
---

工作上電腦多開，常常需要不同電腦間傳資料，這時網路芳鄰就很方便，但有時資料是在Linux和Windows系統之間傳遞，這時就需要在Linux上建立Samba server，讓Windows網路芳鄰可以詢問到Linux資料夾

```
$ sudo apt update
$ sudo apt install samba samba-common
```

編輯Samba配置文件，並在最後一行加入以下內容

```
sudo nano /etc/samba/smb.conf
------------------------------
[sambashare]
	comment = Samba on Ubuntu
	path = /home/kevinlee
	valid users = kevinlee
	read only = No
```

驗證設定檔內容是否正確？

```
$ testparm 
Load smb config files from /etc/samba/smb.conf
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
WARNING: The "syslog" option is deprecated
Processing section "[printers]"
Processing section "[print$]"
Processing section "[sambashare]"
Loaded services file OK.
WARNING: The 'netbios name' is too long (max. 15 chars).

Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions

# Global parameters
[global]
	server string = %h server (Samba, Ubuntu)
	server role = standalone server
	map to guest = Bad User
	obey pam restrictions = Yes
	pam password change = Yes
	passwd program = /usr/bin/passwd %u
	passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
	unix password sync = Yes
	syslog = 0
	log file = /var/log/samba/log.%m
	max log size = 1000
	dns proxy = No
	usershare allow guests = Yes
	panic action = /usr/share/samba/panic-action %d
	idmap config * : backend = tdb


[printers]
	comment = All Printers
	path = /var/spool/samba
	create mask = 0700
	printable = Yes
	browseable = No


[print$]
	comment = Printer Drivers
	path = /var/lib/samba/printers


[sambashare]
	comment = Samba on Ubuntu
	path = /home/kevinlee
	valid users = kevinlee
	read only = No

```

創建Samba帳號密碼

```
$ sudo smbpasswd -a kevinlee
```

Windows連線到Samba server並輸入帳號和密碼

![image-20200508133834726]({{site.baseurl}}/img/image-20200508133834726.png)