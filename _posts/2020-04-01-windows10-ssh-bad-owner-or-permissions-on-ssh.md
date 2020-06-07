---
title:  Windows 10 SSH Bad owner or permissions on .ssh/config
---

最近win10上ssh会出现`Bad owner or permissions on .ssh/config`错误，似乎是由于`config`文件权限不对，可是在尝试修改`.ssh`和`config`文件的所有权和权限后还是不行，最后尝试卸载win10自带的`openssh`客户端重新安装openssh官网的客户端后问题解决了。

<!--description-->

# OpenSSH 卸载

安装卸载参考[openssh_install_firstuse]([https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
)
* 查找win10自带的`OpenSSH`

```powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'

# This should return the following output:

Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent
Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```

* 卸载`OpenSSH`

```powershell
# Uninstall the OpenSSH Client
Remove-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Uninstall the OpenSSH Server
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

* 或者安装`OpenSSH`

```powershell
# Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Both of these should return the following output:

Path          :
Online        : True
RestartNeeded : False
```
# 通过Scoop安装OpenSSH

[scoop](https://scoop.sh/)
```powershell
scoop install openssh
```
