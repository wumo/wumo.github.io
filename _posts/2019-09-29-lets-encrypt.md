+++
title= "Let's Encrypt"
date= "2019-09-29"
tags= ["ca","Let's Encrypt","acme"]
+++
Using acme to manage Let's Encrypt certificate.
<!--more-->

Install [<u>acme</u>](https://github.com/Neilpang/acme.sh).
```shell
curl https://get.acme.sh | sh
```
Issue a certificate (change parameters if you want different certificate)
```shell
acme.sh --issue --dns dns_ali -d example.com -k 4096 --log
```
Install the certificate to specific folder
```shell
acme.sh --installcert --nocron -d example.com --fullchainpath /home/wumo/cert/example.com.crt --keypath /home/wumo/cert/example.com.key --reloadcmd "sudo gitlab-ctl hup nginx"
```
**Note**: change `--fullchainpath` and `--keypath` to your folder. change `--reloadcmd` to your app-reload cmd. 

If the `--reloadcmd` needs `sudo` permission, then you need to remove default user `crontab` job and manaually create a sudo user `crontab` job:
```shell
sudo crontab -e
32 0 * * * "/home/wumo/.acme.sh"/acme.sh --cron --home "/home/wumo/.acme.sh" >> /home/wumo/.acme.sh/cron.log 2>&1

```
