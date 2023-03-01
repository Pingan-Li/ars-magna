# 在Fedora上安装驱动

目前有两种方法，一种是通过Fedora提供的第三方软件仓库进行安装; 第二中是通过NVIDIA CUDA提供的仓库进行安装。
推荐使用第二种方法。

## 方法一

在 Software store 的Preference中加入 RPM Fusion for Fedora 36 - Nonfree - NVIDIA 的第三方软件库
然后：

```shell
sudo dnf install akmod-nvidia
sudo dnf install xorg-x11-drv-nvidia
```

## 方法二

使用CUDA提供的软件库

```shell
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora37/x86_64/cuda-fedora37.repo
sudo dnf clean all
sudo dnf -y module install nvidia-driver:latest-dkms
sudo dnf -y install cuda
```

两种方法尽量不要混用。
