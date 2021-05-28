---
layout: post
title: Lichee ZERO Adventures Part-2
date: 16-05-2021 12:20:00 +0200
description: My Adventure with LicheePi Zero. # Add post description (optional)
img: LicheePi-Zero-page.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux, Lichee ZERO]
---

Compile linux source code and make it blink!  

## 1. Get and Install cross compiler:

* Get the cross compiler
> wget https://releases.linaro.org/components/toolchain/binaries/6.3-2017.05/arm-linux-gnueabihf/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz  
> tar xvf gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz  
> mv gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf /opt/  

* Set the PATH by adding the line to the "bash.bashrc" file.
> sudo /etc/bash.bashrc  

    ``` PATH="$PATH:/opt/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf/bin" ```

## 2. Download and compile linux source code
> git clone https://github.com/Lichee-Pi/linux.git  
> cd linux  
> make ARCH=arm licheepi_zero_defconfig  
> make ARCH=arm menuconfig   #add bluethooth, etc - Optional.  
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j16  
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j16 INSTALL_MOD_PATH=out modules  
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j16 INSTALL_MOD_PATH=out modules_install  

After compiling, zImage is under arch/arm/boot/, and the driver module is under out/

[Lichee ZERO Mainline Kernel basic compilation guide](http://zero.lichee.pro/%E7%B3%BB%E7%BB%9F%E5%BC%80%E5%8F%91/kernel_build.html)  
[Linaro Releases](https://releases.linaro.org/components/toolchain/binaries/latest-5/arm-linux-gnueabihf/)