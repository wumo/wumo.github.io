+++
title= "Install tensorflow and gym"
date= "2018-03-26"
tags= ["drl"]
+++

<!--more-->
# Prerequisite: 
### Install Ubuntu 16.04 with nvidia graphics card
turn off BIOS Secure Boot and Fast Boot

# Install CUDA 9.2
Download latest CUDA Toolkit runfile from [here](https://developer.nvidia.com/cuda-downloads).

The file will be like cuda_9.2.148_396.37_linux.run

`9.2` is the cuda version. `396.37` is the version of the bundled nvidia graphics driver.

Install using `sudo sh cuda_9.2.148_396.37_linux.run`

While installing, you can choose to install the bundled driver.

Or you can install the latest compatible graphics driver from the official nvidia ppa
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install nvidia-396 # should be the same as the bundled driver of cuda.
nvidia-smi # check
```

# Install cuDNN 7.2 for CUDA 9.2
Download [cuDNN](https://developer.nvidia.com/cudnn).
Install according to  [NVIDIA's documentation](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html):
```
tar -xvzf cudnn-9.2-linux-x64-v7.2.1.38.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```
Append following lines to `/etc/environment`:
```
LD_LIBRARY_PATH="/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
CUDA_HOME="/usr/local/cuda"
```

# Download or compile tensorlfow-gpu
We are going to use tensorflow 1.10.1 with cuda 9.2. There is no official released version of tensorflow that supports cuda 9.2, so you can choose to build one yourself or download others' compiled one such as [TensorFlow-wheels](https://github.com/evdcush/TensorFlow-wheels/releases)
```
wget https://github.com/evdcush/TensorFlow-wheels/releases/download/tf-1.10.0-gpu-9.2-tensorrt-mkl/tensorflow-1.10.0-cp36-cp36m-linux_x86_64.whl
```

# Install Python (prefer conda, here using miniconda) and tensorlfow-gpu
```
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
```
remember to export conda path properly (modify `~/.zshrc` if you're using zsh)

create an environment that is compatible to your compiled or downloaded tensorlfow wheel.
```
conda create -n tensorflow-gpu
source activate tensorflow-gpu
```
Because `tensorflow-1.10.0-cp36-cp36m-linux_x86_64.whl` requires python 3.6, we need to install one.
```
conda install python=3.6 # 
```
Install `tensorlfow-gpu`
```
pip install tensorflow-1.10.0-cp36-cp36m-linux_x86_64.whl
```
Test if install
```
python
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
```

# Install OpenAI Gym/baselines

```
git clone https://github.com/openai/mujoco-py.git
cd mujoco-py
pip install -e .
cd ..
sudo apt install cmake libopenmpi-dev zlib1g-dev
git clone https://github.com/openai/baselines.git
cd baselines
pip install -e .
# If you experience too long time to setup `mujoco`, try using proxy to `pip install`, e.g. proxychains pip install -e .
```

# *Notes
### (Optional)Using thrid-party `ubuntu` repository

Replace the content of `/etc/apt/sources.list` to the below:
```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
```
then 
```
sudo apt update
sudo apt upgrade
```

### (Optional)Using thrid-party `pip` repository
Create/Modify `~/.pip/pip.conf` to use third party mirror:
```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

### (Optional)Using Shadowsocksrr to run command through proxy
```
sudo apt install git
git clone https://github.com/shadowsocksrr/shadowsocksr.git
cd shadowsocksr
bash initcfg.sh
nano user-config.json # server address etc.
sudo nano /lib/systemd/system/shadowsocksr.service
```
Append following lines to `/lib/systemd/system/shadowsocksr.service`:
```
[Unit]
Description=Shadowsocks Client Service
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/bin/python /home/[username]/shadowsocksr/shadowsocks/local.py -c /home/[username]/shadowsocksr/user-config.json

[Install]
WantedBy=multi-user.target
```
**Change** the `[username]` to your user name.
```
sudo systemctl enable shadowsocksr.service
sudo systemctl start shadowsocksr.service
```
Then install `proxychains` to proxy any command through socks5.
```
sudo apt-get install proxychains
```
Edit proxychains config file /etc/proxychains.conf
```
...
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5  127.0.0.1 1080
```
Run like this:
```
proxychains git clone ...
```
Also you can install `polipo` to turn socks5 into a http proxy. Then export environment variable `http_proxy`,`https_proxy` and `no_proxy` to enable global http proxy.

