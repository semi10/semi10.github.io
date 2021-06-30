---
layout: post
title: Lichee ZERO Adventures Part-3
date: 28-05-2021 18:00:00 +0200
description: My Adventure with LicheePi Zero. # Add post description (optional)
img: LicheePi-Zero-page.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Linux, Lichee ZERO]
---

Back to the Basic!  

## 1. Linux Boot Requirements:
### RBL: ROM Boot Loader - Runs out of ROM.  
* Stack Setup
* Watchdog timer 1 configuration (set to 3 minutes timeout)
* System clock configuration using PLL
* Search Memory devices or other bootable interfaces for MLO/SPL
* Copy MLO/SPL in to the internal SRAM of the chip
* Execute MLO/SPL

### SPL/MLO: Secondary Program Loader/Memory Loader - Runs out of Internal SRAM.
* UART console initialization to print out the debug messages.
* Reconfigures the PLL to desired value.
* Initializes the DDR registers to use the DDR memory.
* Does muxing configurations of boot peripherals pin, because its next job is to load the U-boot from the boot peripherals.

### U-boot - Runs out of DDR.
* Initialize some of the peripherals like, I2C, NAND, FLASH, ETHERNET, UART, USB, MMC, etc.
* Load the Linux kernel image from various boot sources to the DDR memory of the board.
* Boot sources: USB, eMMC, SD card, Ethernet, Serial, NAND flash, etc.
* Passing of boot arguments to the kernel.
Boot behavior of the U-boot can be changed by using uEnv.txt which set the env variables that drives the U-boot.
UImage = U-boot+zImage
DTB = Device Tree Binary

### Linux Kernel - Runs out of DDR.
### RFS: Root File System - SD/Flash/Network/RAM/e-MMC.

