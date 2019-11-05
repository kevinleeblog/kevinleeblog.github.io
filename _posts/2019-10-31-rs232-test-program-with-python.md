---
layout: post
title: "RS232 test program with python"
auther: Kevin Lee
category: project1
tags: [Linux, Job_Logging, Python]
subtitle:
visualworkflow: true
---

### 為何做？

繼上篇[OpenSUSE/SUSE Network Configuration](../opensusesuse-network-configuration)
處理好了網路問題後，QA反應不知道如何去測試RS232 Port

所以我問了一下以前是怎麼去測試，說是安裝minicom後在把Tx與Rx對接
思索一下後和主管建議我可以額外去寫一隻RS232的產測程式，得到雙手雙腳贊成後
就開始思考怎麼寫

我公司的產品使用Linux及Windows都有可能，所以想寫一隻可以跨平台的程式，就決定使用Python


### 如何做？

#### Ubuntu-16.04 OS

安裝Serial Library for Python

```
$ sudo apt-get install python-serial
```

#### Windows OS

Download and Install Python2.7
https://www.python.org/downloads/release/python-2717/

Download and Install pip
https://bootstrap.pypa.io/get-pip.py
打開Windows CMD，安裝pip

```
$ python get-pip.py
$ pip -v
```

安裝pySerial

`$ pip install pyserial`

#### Uart loopback Test程式

指令下法：

##### Linux

`uart_test.py /dev/ttyUSB0`

##### Windows

`uart_test.py COM1`

說明

使用前的Comport Tx與Rx要對接

![20191101_105945]({{site.baseurl}}/img/20191101_105945.jpg)

程式會從Tx送0~9給Rx
然後比對送和收的數值是否相等來判定是否通過

```python
import serial
import sys

if __name__ == '__main__':
	if len(sys.argv) != 2:
		print('Usage: uart_test.py <RS232 COMPORT>')
		sys.exit(-1)
		
	ComPort=sys.argv[1]
	mySerial = serial.Serial(ComPort, 115200, timeout=1)	
	
	for num in range(10):
		sendData=bytes([num])
		result=mySerial.write(sendData)
		recvData=mySerial.readline()
		if sendData!=recvData:
			print('Test Fail!')
			sys.exit(-1)
		print('.')
	
	print('Test OK!')
	sys.exit(0)
```

執行結果

Linux OS

```
$ sudo python uart_test.py /dev/ttyUSB0
.
.
.
.
.
.
.
.
.
.
Test OK!
```

Windows OS

![image-20191101134116985]({{site.baseurl}}/img/image-20191101134116985.png)

> 目前程式的處理送和收的反應比較慢，還沒有優化