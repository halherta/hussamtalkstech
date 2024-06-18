+++
author = "Hussam Al-Hertani"
title = "Flashing Micropython on the ESP32-S3 Microcontroller: Part 2"
date = "2024-06-17"
draft = false
description = "ESP32S3 in micropython"
[params]
  math = true
tags = [
    "Micropython", "ESP32"
]
categories = [
    "Micropython", "ESP32"
]
+++

I recently purchase an [ESP32-S3-DevKitC-1 v1.1](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/hw-reference/esp32s3/user-guide-devkitc-1.html) development board. I'm well versed in C but wanted to program this particular board in Micropython instead.

I got the *ESP32-S3-DevKitC-1-N16R8V* variant with the [ESP32-S3-WROOM-2-N16R8V](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-2_datasheet_en.pdf) module soldered on. This particular module comes with 16MB of octal SPI Flash memory and 8MB of octal SPI PSRAM; plenty of program memory and RAM for all of those Micropython objects!.

To get started, the latest precompiled Micropython binary can be downloaded from the [official Micropython download page](https://micropython.org/download/ESP32_GENERIC_S3/). I downloaded the latest Micropython build  (.bin format) which was the:  [*ESP32_GENERIC_S3-SPIRAM_OCT-20240602-v1.23.0.bin*](https://micropython.org/resources/firmware/ESP32_GENERIC_S3-SPIRAM_OCT-20240602-v1.23.0.bin). Be sure to get the binary with Octal-SPIRAM support.

```bash
:~$ wget https://micropython.org/resources/firmware/ESP32_GENERIC_S3-SPIRAM_OCT-20240602-v1.23.0.bin
```

Once downloaded, the next step is to *flash* the image onto the board using the esp-tool programmer. To install it, let's first create a new virtual python environment. This looks like the following on Debian Bookworm: 

``` Bash
:~$ mkdir -p ~/Development/PythonVirtEnvironments/
:~$ python3 -m venv ~/Development/PythonVirtEnvironments/EsptoolEnv
```

Now activate the Python virtual environment and install the esptool.

```Bash
:~$ source ~/Development/PythonVirtEnvironments/EsptoolEnv/bin/activate
(EsptoolEnv) :~$ pip install esptool
```

To flash the Micropython bin file, connect the USB cable to the USB port on the left of the board (the one attached to the UART0 peripheral on the ESP32S3's via the USB-Serial FTDI chip). Make sure that you are using the right port name. In my case that was `/dev/ttyUSB0`. Run the following commands to: i) erase the  ESP32S3 module, & ii) program it with the downloaded binary:

```Bash 
(EsptoolEnv) :~$ esptool.py --chip esp32s3 --port /dev/ttyUSB0 erase_flash
(EsptoolEnv) :~$ esptool.py --chip esp32s3 --port /dev/ttyUSB0 write_flash -z 0 ESP32_GENERIC_S3-SPIRAM_OCT-20240602-v1.23.0.bin
```
You should get the following output:

![**Figure 1. Flashing Micropython on the EESP32-S3-DevKitC-1 v1.1 board**](fig1.png "700px")


Now install and run Thonny; an easy to use yet versatile Micropython IDE:

```Bash
(EsptoolEnv) :~$ pip install thonny
(EsptoolEnv) :~$ thonny
```
Goto `Run->Configure Interpreter` and ensure that the `Interpreter`tab in the `Thonny options` look as shown below: 

![**Figure 2. Thonny Interpreter settings**](fig2.png "700px")


click OK on the dialog and you should be presented with the Micropython shell at the bottom of the thonny window as shown in the figure below.

![**Figure 3. Micropython shell ready!**](fig3.png "700px")


To get started, let's see how much PSRAM and Flash are detected in the Micropython shell. To find out how much PSRAM is detected, type the following:

```Python
import gc

gc.mem_free()
```
![**Figure 4. PSRAM available**](fig4.png "700px")

Notice that we get `8312160` bytes. This is close enough to the 8MB of PSRAM that we were expecting (`8388608` bytes). The difference is very likely the memory used to launch the Micropython interpreter.

Now what about Flash ? 

```Python
import esp

esp.flash_size()
```
![**Figure 5. FLASH available**](fig5.png "700px")

For Flash, it looks like Micropython is only able to detect about 8MB of the 16MB available Flash. The Micropython `ESP32_GENERIC_S3` images all seem to support a maximum of 8MB even if the on module Flash memory is more. That's a bummer!  I've seen ESP32-S3 boards with as much as 32MB of Flash being sold on Digikey. To be fair, if you purchase the **ESP32-S3-DevKitC-1-N8R8V** variant of the ESP32-S3 Devkit board (with 8MB of Flash and 8MB of PSRAM), this image will be all you need. 

While the prebuilt Micropython build was successfully installed onto the ESP32-S3 board and is fully functional, the fact that only half the available Flash is detected annoys me a little. In a future entry, I'll demonstrate how to build a Micropython image from source that supports the 16MB of Flash in its entirety.