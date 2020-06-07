---
title= "Shadowsocks的一些常见用法"
tags:
- shadowsocks
---

## Install Shadowsocks On Ubuntu


```bash
sudo apt-get install python-pip
sudo pip install shadowsocks
```

check the installation of `shadowsocks`:

```bash
where sslocal
/usr/local/bin/sslocal
/usr/local/bin/sslocal

where ssserver
/usr/local/bin/ssserver
/usr/local/bin/ssserver
```
## Simple Usage

Touch a config file `config.json` as follows:
```json
 {                               
      "server": "server ip or server domain",
      "server_port": port only for server,
      "local_address": "127.0.0.1",
      "local_port" : port only for client,
      "password": "password",  
      "method": "rc4-md5",       
      "remarks": "",             
      "auth": false,             
      "timeout": 300             
}                                
```
Default local address is `127.0.0.1`. Dfault local port is `1080`.
Run as shadowsocks server:
```bash
sudo ssserver -c config.json -d start
```
Run as shadowsocks client:
```bash
sudo sslocal -c config.json -d start
```
<!--more-->

## Start Shadowsocks Automatically On Ubuntu 16.10 Using Systemd

ref: [Start Shadowsocks Automatically On Ubuntu 16.10 Using Systemd](https://searene.github.io/2016/12/06/Start-Shadowsocks-Automatically-On-Ubuntu-16-10-Using-Systemd/)

Create a new file called `shadowsocks.service` in `/lib/systemd/system`:

```toml
[Unit]
Description=Shadowsocks Client Service
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/local/bin/sslocal -c /home/wumo/shadowsocks/config.json

[Install]
WantedBy=multi-user.target
```
Note that `/usr/local/bin/sslocal -c /home/wumo/shadowsocks/config.json` is the command to be executed after network is on. 
Change `/home/wumo//shadowsocks/config.json` to your shadowsocks config file.

Start, restart or stop shadowsocks service:
```bash
sudo systemctl start shadowsocks.service
sudo systemctl start shadowsocks
sudo systemctl restart shadowsocks
sudo systemctl stop shadowsocks
```
check the status of shadowsocks service:
```bash
sudo systemctl status shadowsocks
● shadowsocks.service - Shadowsocks Client Service
   Loaded: loaded (/lib/systemd/system/shadowsocks.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2017-04-02 12:28:06 CST; 1min 41s ago
 Main PID: 22037 (sslocal)
    Tasks: 1
   Memory: 15.5M
      CPU: 161ms
   CGroup: /system.slice/shadowsocks.service
           └─22037 /usr/bin/python /usr/local/bin/sslocal -c /home/wumo/shadowsocks/config.json

Apr 02 12:28:06 wumo-pc systemd[1]: Started Shadowsocks Client Service.
Apr 02 12:28:06 wumo-pc sslocal[22037]: INFO: loading config from /home/wumo/shadowsocks/config.json
Apr 02 12:28:07 wumo-pc sslocal[22037]: 2017-04-02 12:28:07 INFO     loading libcrypto from libcrypto.so.1.0.0
Apr 02 12:28:07 wumo-pc sslocal[22037]: 2017-04-02 12:28:07 INFO     starting local at 127.0.0.1:1080
```
Enable auto start
```bash
sudo systemctl enable shadowsocks.service
```
## Start Shadowsocks Automatically On Ubuntu 14.04 Using Upstart

Go to /etc/init/
```bash
sudo nano shadowsocks.conf
```
write the following content:
```
description "Shadowsocks Server"
author      "wumo"

# used to be: start on startup

start on started networking
stop on shutdown

# automatically respawn

respawn
respawn limit 99 5

script

    exec /usr/bin/python /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start

end script

post-start script

   # optionally put a script here that will notifiy you node has (re)started
   # /root/bin/hoptoad.sh "node.js has started!"

end script
```
start shadowsocks manually:
```bash
sudo start shadowsocks
```
And when you wanna kill the process `sudo stop shadowsocks`(seemed not working)

## Turn Shadowsock into Http proxy

Install `polipo`
```bash
sudo apt-get install polipo
```

Edit the config file of polipo `/etc/polipo/config`
```
# This file only needs to list configuration variables that deviate
# from the default values.  See /usr/share/doc/polipo/examples/config.sample
# and "polipo -v" for variables you can tweak and further information.

logSyslog = true
logFile = /var/log/polipo/polipo.log

proxyAddress = "0.0.0.0"

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5

chunkHighMark = 50331648
objectHighMark = 16384

serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
```
Restart `polipo` service
```bash
sudo systemctl restart polipo
```
The default port of `polipo` is `8123`.
You can `export http_proxy` to make it gloabl http proxy
```
export http_proxy="http://127.0.0.1:8123/"
export https_proxy="http://127.0.0.1:8123/"
```

## Execute command by socks proxy

Install `proxychains`
```bash
sudo apt-get install proxychains
```
Edit `proxychains` config file `/etc/proxychains.conf`
```
...
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 1080
```

Then you can run almost every command through socks proxy by :
```bash
proxychains wget ...
```

## Reference

more on [shadowsocks/wiki](https://github.com/shadowsocks/shadowsocks/wiki)