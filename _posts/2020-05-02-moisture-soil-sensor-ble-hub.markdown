---
layout: post
title: Moisture Soil Sensor BLE Hub
date: 02/05/2020 18:26:00 +0200
description: Moisture Soil Sensor Hub Using cc2541 BLE module and Android App. # Add post description (optional)
img: moisture-soil-sensor-ble-hub-page.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Android, BLE, CC2541]
---
The goal othe project is to monitor Soil Moisture level of multiple (up to- !) flowerpots. The information will be shown by Android app.

## Step 1: Check your BLE module
To read analog signal of Capacitive Soil Moisture Sensors I use CC2541 BLE module. Surprisingly we don't need any additional microcontroller or custom code.
Unfortunately to make it work we need original HM-10 BLE module or we can change the firmware of Clone HM-10 BLE Module.
The rule of thumb is that if your module has only one crystal oscillator it is not original HM-10 BLE Module.
Here is a link to excellent tutorial that will help you to [Identify HM-10 BLE Module]. 
If you have original module you can skip to the next Step.

## Step 2: Solder
* To burn the bootloader we need to solder wires to get access to Debug_CLK (P2_2), Debug_Data (P2_1), RESETB, VCC and GND pins.
* To upload the firmware we need to solder additional wires for UART_TX and UART_RX pins.

![HM-10 Soldered]({{site.baseurl}}/assets/img/Soldering2541.jpg)

## Step 3: Burn the bootloader
* In order to burn the firmware download [folowing software]
* Extract the zip file
* Upload the CCloaderArduino.ino sketch to Arduino Uno it enables us to burn original firmware to BLE module in the next step.
* Connect BLE module to arduino board as described below:

| Arduino | HM-10      |    
|---------|------------|
| 6       | DEBUG_DATA |
| 5       | DEBUG_CLK  |
| 4       | RESETB     |
| 3V3     | VCC        |
| GND     | GND        |

* Open command prompt (cmd) and navigate to extracted previosly folder containing CCloader.exe
* Execute following command:  
  > CCLoader.exe \<Arduino COM Port\> \<Firmware File\> 0  

  In My case The command was:  

  > CCLoader.exe 1 CC2541hm10v540.bin 0

  The End of the process should be look like:

  ![Firmware Uploaded]({{site.baseurl}}/assets/img/FirmwareUploaded.jpg)






## Step 2: Configure BLE module


[Identify and Flash the Firmware on Clone HM-10 BLE Module]: https://circuitdigest.com/microcontroller-projects/how-to-flash-the-firmware-on-cloned-hm-10-ble-module-using-arduino-uno
[folowing software]: https://circuitdigest.com/sites/default/files/Flashing-firmware-on-HM10-BLE-Module.zip
[fresh firmware]: http://www.jnhuamao.cn/download_rom_en.asp?id=1