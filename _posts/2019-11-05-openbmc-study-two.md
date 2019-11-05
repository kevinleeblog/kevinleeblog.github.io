---
layout: post
title: "OpenBMC Basic-OpenBMC SDK(二)"
auther: Kevin Lee
category: project1
tags: [OpenBMC]
subtitle:
visualworkflow: true
---



### OpenBMC SDK有什麼用途？

想像一個狀況，如果今天有個Demo Code想要放在OpenBMC上去執行，需要那些步驟？

首先，你將需要為這個Demo Code寫一個Yocto Recipe，同時還要指定這個Demo Code是要當成Library? Daemon？這會影響到Recipe調用的參數不同

那有沒有更快的方法，我先在本機編譯好後再透過網路丟到OpenBMC上看結果？有的，這就是OpenBMC SDK的用途，講白一點SDK會把編譯OpenBMC所使用的Library和Compile給放到環境變數內，給使用者直接去使用。

### SDK安裝

執行指令前，先確認先前已經編譯完成並產生image

```
$ cd openbmc
$ . openbmc-env
$ bitbake -c populate_sdk obmc-phosphor-image
```

編譯一會兒後，產生sdk目錄及Script，執行Script

```
$ tmp/deploy/sdk/oecore-x86_64-arm1176jzs-toolchain-nodistro.0.sh

Phosphor OpenBMC (Phosphor OpenBMC Project Reference Distro) SDK installer version nodistro.0
=============================================================================================
You are about to install the SDK to "/usr/local/oecore-x86_64". Proceed [Y/n]? Extracting SDK..........................................................................................done
Setting it up...done
SDK has been successfully set up and is ready to be used.
Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
 $ . /usr/local/oecore-x86_64/environment-setup-arm1176jzs-openbmc-linux-gnueabi
```

以後要使用sdk前先執行

```
$ . /usr/local/oecore-x86_64/environment-setup-arm1176jzs-openbmc-linux-gnueabi
```

### SDK使用

舉例來說，有個Demo Code執行會輸出*"Hello!OpenBMC"*
demo.c

```c
#include <stdio.h>

int main()
{
	printf("Hello!OpenBMC\n");
	return 0;
}
```

編譯

```
$ $CC demo.c -o my_demo
```

傳送給Qemu (Qemu要已經啟動)

```
$ scp -P 2222 my_demo root@127.0.0.1:/tmp
```

在Qemu內執行demo

```
root@romulus:/tmp# cd /tmp
root@romulus:/tmp# ./my_demo 
Hello!OpenBMC
root@romulus:/tmp#
```

