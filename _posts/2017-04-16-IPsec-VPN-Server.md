---
title= "Set Up IPsec VPN Server"
tags:
- vpn
---
<!--more-->

ref [IPsec VPN Server Auto Setup Scripts](https://github.com/hwdsl2/setup-ipsec-vpn)

```bash
wget https://git.io/vpnsetup -O vpnsetup.sh
nano vpnsetup.sh
[Replace with your own values: YOUR_IPSEC_PSK, YOUR_USERNAME and YOUR_PASSWORD]
sudo sh vpnsetup.sh
```

Settings in Windows 10

1. Right-click on the wireless/network icon in your system tray.
2. Select **Open Network and Sharing Center**.
3. Click **Set up a new connection or network**.
4. Select **Connect to a workplace** and click **Next**.
5. Click **Use my Internet connection (VPN)**.
6. Enter Your VPN Server IP in the **Internet address** field.
7. Enter anything you like in the **Destination name** field, and then click **Create**.
8. Return to **Network and Sharing Center**. On the left, click **Change adapter settings**.
9. Right-click on the new VPN entry and choose **Properties**.
10. Click the **Security** tab. Select "Layer 2 Tunneling Protocol with IPsec (L2TP/IPSec)" for the **Type of VPN**.
11. Click **Allow these protocols**. Be sure to select the "Challenge Handshake Authentication Protocol (CHAP)" checkbox.
12. Click the **Advanced settings** button.
13. Select **Use preshared key for authentication** and enter Your VPN IPsec PSK for the **Key**.
14. Click **OK** to close the **Advanced settings**.
15. Click **OK** to save the VPN connection details.

Note: A one-time registry change is required before connecting. See details below.

Windows Error 809
> The network connection between your computer and the VPN server could not be established because the remote server is not responding.

to fix this error, a one-time registry change is required because the VPN server and/or client is behind NAT (e.g. home router). 
Refer to the linked web page, or run the following from an elevated command prompt. When finished, reboot your PC.
For Windows Vista, 7, 8.x and 10
```cmd
REG ADD HKLM\SYSTEM\CurrentControlSet\Services\PolicyAgent /v AssumeUDPEncapsulationContextOnSendRule /t REG_DWORD /d 0x2 /f
```

**Restart**