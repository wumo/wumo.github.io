---
title:  Offline Install Msys2 Packages
---

<!--description-->

# Install Msys2 with Internet

You have to install msys2 and your needed packages first. Check [msys2.org]([https://www.msys2.org/](https://www.msys2.org/)
)

# Extract dependent packages

To extract dependent packages required by your need packages, run the following command in the msys2 terminal: [#Recursively list dependencies of a package in arch linux
](https://superuser.com/questions/1308034/recursively-list-dependencies-of-a-package-in-arch-linux)
```bash
echo "package_name1 package_name2 ..." | xargs -n 1 pactree -u | sort -u | xargs -n 1 pacman -Sp > /myPackages.list
```
Replace `package_name1 package_name2 ...` with your needed packages. 

# Scoop support zstd

[Support zstd for extraction](https://github.com/lukesampson/scoop/issues/3990)
