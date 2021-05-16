---
layout: post
title: Lichee ZERO Adventures
date: 16-05-2021 9:30:00 +0200
description: My Adventure with LicheePi Zero. # Add post description (optional)
img: LicheePi-Zero-page.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux]
---

How to create bootable SDcard and turn on the screen.  


## 1. Boot the Lichee ZERO:
* Dowload one of the available images located in the "9.[Downloads]" section of the [LICHEE PI ZERO page][0].  
The image I choose is [lichee_zero_test_Debian_LXDE][1].  
* Extract the lichee_zero-Debian-LXDE_800_alpha.dd file and change the name extension from .dd to .img it will enable [Win32 Disk Imager][2] to recognize the file as image.  
* Write the image to the SD card using the Win32 Disk Imager mentioned above. Ensure that you selected the right Device (SD card letter).  
  
If all went ok you should now be able to insert the SD card to the Lichee Zero connect it to the power and boot successfully.

## 2. Downolad and compile U-Boot to support 4.3-inch screen
By default Lichee ZERO supports 5-inch screen which was too large for me. In order to fix the flickering issue on 4.3-inch screen it requires to recompile and rewrite the boot to support proper 4.3-inch screen resolution.  

* Clone and enter U-Boot repository with:
> git clone https://github.com/Lichee-Pi/u-boot -b v3s-current  
> cd u-boot
* Configure for 480x272 (4.3-inch screen). Additional configurations can be found in configs directory such as  "LicheePi_Zero_800x480LCD_defconfig" and "LicheePi_Zero_defconfig".  
> make ARCH=arm LicheePi_Zero_480x272LCD_defconfig
* Compile
> make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4
* If the compilation was sucssessfull you should be able to locate the created u-boot-sunxi-with-spl.bin in the u-boot folder.
You should check what is the mounted directory of your SD card using:
>df -h
* Now you can write the generated file with 8K offset to boot of SD card (in my case it is /dev/sdb). 
> sudo dd if=u-boot-sunxi-with-spl.bin of=/dev/sdb bs=1024 seek=8

|Before                                                                                         |After                                                                    |
|-----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|
|![Lychee-Zero-Flickering-Screen]({{site.baseurl}}/assets/img/Lychee-Zero-Flickering-Screen.gif)|![Lychee-Zero-Screen]({{site.baseurl}}/assets/img/Lychee-Zero-Screen.gif)|


[Additional information about U-Boot Compilation][3]

## 3. Enable auto-login in Debian Xfce
* Edit 01_debian.conf file:
> sudo nano /usr/share/lightdm/lightdm.conf.d/01_debian.conf
* Add following lines to the file:
```
autologin-user=username  
autologin-user-timeout=0
```
Don't forget to save the file before exit. 

## 4. Disable touchscreen
* Edit 10-evdev.conf file:
> sudo nano /usr/share/X11/xorg.conf.d/10-evdev.conf
* Add following lines to the file:
```
Section "InputClass"
        Identifier "evdev touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "evdev"
        Option "Ignore" "on" # Disable device"
EndSection
```



[0]: https://licheepizero.us/
[1]: https://licheepizero.us/downloads/lichee_zero_test_Debian_LXDE.tar.bz2
[2]: https://sourceforge.net/projects/win32diskimager/
[3]: https://licheepizero.us/build--uboot-for-licheepi-zero