---
title= "Install KMS on Ubuntu"
tags: 
- kms
---

ref [Jalena Blog - Linux KMS服务器](https://jalena.bcsytv.com/archives/1388)


```bash
#download kms files from http://forums.mydigitallife.info/threads/50234-Emulated-KMS-Servers-on-non-Windows-platforms
sudo apt-get install p7zip-full # to unzip 7z file
mkdir kms
cd kms
7z x vlmcsd*****
sudo mv vlmcsd/binaries/Linux/intel/static/vlmcsd-x64-musl-static /usr/local/bin/kms
#grant permission
sudo chmod u+x /usr/local/bin/kms
#run
/usr/local/bin/kms
#check, should run on port 1688
netstat -tlnp
```
added to service
```
[Unit]
Description=KMS Server
After=network.target

[Service]
Type=forking
PIDFile=/var/run/kms.pid
ExecStart=/usr/local/bin/kms -p /var/run/kms.pid
ExecStop=/bin/kill -HUP $MAINPID
PrivateTmp=True
Restart=always

[Install]
WantedBy=multi-user.target
```
On windows
run powershell in administrator mode:
```bash
PS C:\WINDOWS\system32> slmgr.vbs -upk
PS C:\WINDOWS\system32> slmgr.vbs -ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
PS C:\WINDOWS\system32> slmgr.vbs -skms kms.bcsytv.com
PS C:\WINDOWS\system32> slmgr.vbs -ato
PS C:\WINDOWS\system32> slmgr.vbs -dlv
```
|version|key|
|----|----|
|Windows 10 Professional|W269N-WFGWX-YVC9B-4J6C9-T83GX|
|Windows 10 Enterprise|NPPR9-FWDCX-D2C8J-H872K-2YT43|
|Windows 10 Education|NW6C2-QMPVW-D7KKK-3GKT6-VCFB2|

make kms a ubuntu service 

Download [**vlmcsd-1108-2017-01-19-Hotbird64.7z**](https://gitlab.com/wumo/wumo.gitlab.io/raw/master/static/vlmcsd-1108-2017-01-19-Hotbird64.7z)

<!--more-->