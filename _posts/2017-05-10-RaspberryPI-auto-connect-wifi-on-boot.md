---
title:  "Raspberry PI auto connect wifi on boot"
tags:  ["raspberry pi","wifi"]
---
<!--more-->

modify `/etc/network/interfaces`:
```
auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp

```

config in `/etc/wpa_supplicant/wpa_supplicant.conf`:
```
...
network={
        ssid="MARS"
        psk="*************"
        key_mgmt=WPA-PSK
}
```