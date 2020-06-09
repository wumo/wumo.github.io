---
title: "Nsight Graphics On Ubuntu 20.04"
tags: 
- ubuntu
- nsight graphics
---


# Create desktop file for nsight graphics

Create `nsight-graphics.desktop` as:

```
[Desktop Entry]
Version=1.0
Type=Application
Name=Nsight Graphics
Icon=/home/wumo/Pictures/nvidia.png
Exec=/bin/sh /home/wumo/nvidia/NVIDIA-Nsight-Graphics-2020.3/host/linux-desktop-nomad-x64/ngfx-ui
Categories=Development;IDE;
Terminal=false
```

where `Exec=/bin/sh /home/wumo/nvidia/NVIDIA-Nsight-Graphics-2020.3/host/linux-desktop-nomad-x64/ngfx-ui` is the location of shell script of `nsight grphics`. `Icon=/home/wumo/Pictures/nvidia.png` can be any picture you have locally.

# Solve ERR_NVGPUCTRPERM: Permission issue with Performance Counters

Do as [ERR_NVGPUCTRPERM-permission-issue-performance-counters](https://developer.nvidia.com/nvidia-development-tools-solutions-ERR_NVGPUCTRPERM-permission-issue-performance-counters#SolnTag):

Create `nsight.conf` in `/etc/modprobe.d` with content:
```
options nvidia "NVreg_RestrictProfilingToAdminUsers=0
```

Restart.