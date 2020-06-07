---
title: "Things You Do After Installing Ubuntu"
tags:
- ubuntu
---

## Create new user

add new user
```bash
sudo adduser wumo
sudo adduser wumo sudo # add user to sudo group
```

## Change Hostname

```bash
sudo hostname wumo-server
sudo nano /etc/hostname
sudo nano /etc/hosts
```

## Install shadowsocks
参见[Shadowsocks的一些常见用法](https://wumo.gitlab.io/post/2017-04-02-shadowsocks-practice/)

## install oh-my-zsh
install `zsh`
```bash
sudo apt-get install zsh
zsh --version
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

<!--more-->