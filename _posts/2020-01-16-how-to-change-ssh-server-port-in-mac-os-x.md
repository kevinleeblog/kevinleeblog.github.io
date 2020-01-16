---
layout: post
title: "How to change ssh server port in mac os x"
auther: Kevin Lee
category: 
tags: [ssh]
subtitle:
visualworkflow: true
---

### 為何做？

因為有時候想要連回公司繼續處理一些公事，但礙於公司有檔Port，目前已知80 port是可用的，所以要把ssh port 22改成80

### 如何做？

1. 打開Terminal

2. sudo vim /etc/services
   搜尋ssh，把port 22改成80

   ```
      77 ssh              80/udp     # SSH Remote Login Protocol
      78 ssh              80/tcp     # SSH Remote Login Protocol
   ```

   

3. Restart ssh server

   ```
   $ sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
   $ sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
   ```

4. Test

   ```
   $ ssh localhost -p 80
   ```

   

