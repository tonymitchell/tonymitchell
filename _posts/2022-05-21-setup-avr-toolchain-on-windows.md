---
title: How to setup the AVR toolchain on Windows
date: 2022-05-21 10:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, avrdude, avr-gcc]
---

> ## Tutorial Versions
> 
> I have written 3 different versions of this tutorial that cover the following different setups:
> 1. **Windows-based** (*this guide*)
> 
>    This options primarily uses Windows-based tools and has two variations depending on how you run make
>    - Windows + GnuWin32
>    - Windows + WSL
>
> 2. [WSL-based]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})
>
>    This options primarily uses WSL-based linux tools and has two variations depending on how you program the chip -- either WSL-based avrdude using usbip to access the USB programmer or Windows-based avrdude to natively access the USB programmer
>    - WSL + usbip
>    - WSL + Windows
>
>    See [How to setup the AVR toolchain on WSL]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})
>
> 3. [Linux-based]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})
>
>    This option uses all native linux-based tools on a native Linux installation.
> 
>    See [How to setup the AVR toolchain on Linux]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})
> 
> 
> You can see a summary of how the versions differ in the table below:
> |Version|avr toolchain|make|avrdude|usb access|
> |-|:-:|:-:|:-:|:-:|
> |Windows+GnuWin32|windows|gnuwin32|windows|-|
> |Windows+WSL|windows|WSL|windows|-|
> |WSL+usbip|WSL|WSL|WSL|usbip|
> |WSL+Windows|WSL|WSL|windows|-|
> |Linux|linux|linux|linux|-|
> 

## Overview

This guide will help you get your environment set up to build projects on the Atmel AVR chips (e.g. ATmega328, ATtiny85, etc.) projects on Windows using Windows-based tools.


## In Windows

Steps:
1. Install AVR toolchain (to build the code)  (OR install Atmel Studio)
2. Install GnuWin32 (for supporting build tools like Make)
3. Install AVR Dude (to flash the chip)
4. Install VSCode (as an IDE) [Optional]

### Download AVR toolchain

#### Download
1. Browse to https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers
2. Find and download the appropriate toolchain for your chip and dev OS. (e.g. AVR 8-bit Toolchain v3.62 – Windows)
    - AVR 8-bit == chips like ATmega328, ATtiny85, etc.
    - You will have to sign up for a free account to complete the download
3. Find and read the Release Notes for your toolchain (if any).
    - e.g. AVR 8-bit Toolchain 3.6.1 - Release Note
    - Can contain important installation instructions (like the fact that you may need to download chip packs)
4. [Optional] IF your chip isn't supported out of the box by the main toolchain, browse to: http://packs.download.atmel.com/ to find and download the appropriate chip packs. Note: most of the popular chips should already be supported).
    - Note that even if you don't NEED the chip packs, there is additional documentation in them that you might find valuable to read.
    - The chip pack files end in .atpack, but are .zip files and need to be expanded to be used.
    - To use a chip pack, you will need to reference the directory via -B and -I options when compiling
        - Example: avr-gcc -mmcu=atmega328pb -B /home/packs/Atmel.ATmega_DFP.1.0.86/gcc/dev/atmega328pb/ -I /home/packs/Atmel.ATmega_DFP.1.0.86/include/

#### Install

1. Create a folder to hold all your dev tools (e.g. C:\DevTools or C:\Users\<user>\DevTools)
2. Unzip downloaded toolchain into folder. Rename folder as desired (e.g. avr8-gnu-toolchain)
    - e.g. Unzipping avr8-gnu-toolchain-3.6.2.1778-win32.any.x86.zip contained avr8-gnu-toolchain-win32_x86 directory which I renamed to a platform agnostic name of avr8-gnu-toolchain so that any scripts, etc. I create can be cross platform.
3. Unzip avrdude to 


|Zip|Install|Target Location|
|-|-|-|
|avr8-gnu-toolchain-3.6.2.1778-win32.any.x86.zip|Unzip to target|C:\DEV\avr8-gnu-toolchain|
|GetGnuWin32-0.6.3.exe|Run installer, and complete steps in Readme.txt to produce gnuwin32 folder. At a high-level, those steps are: Run download.bat, the run install.bat, then copy folder, then run update-links.bat in the target.|C:\DEV\gnuwin32|
|avrdude-6.4-mingw32.zip|Unzip to target|C:\DEV\avrdude-6.4-mingw32|
|Chip packs (*.atpack)|Unzip to target|C:\DEV\avr8-dvp|


The first few levels of the target structure should look like (note: the avr8-dvp is optional and in this example has several chip packs installed):
```
C:\Dev
├───avr8-dvp
│   ├───atdf
│   ├───avrasm
│   ├───gcc
│   ├───include
│   ├───samd21a
│   ├───samd21b
│   ├───samd21c
│   ├───samd21d
│   ├───samd51a
│   ├───scripts
│   ├───simulator
│   ├───templates
│   └───xc8
├───avr8-gnu-toolchain
│   ├───avr
│   ├───bin
│   ├───doc
│   ├───i686-w64-mingw32
│   ├───info
│   ├───lib
│   ├───libexec
│   ├───man
│   └───share
├───avrdude-6.4-mingw32
└───gnuwin32
    ├───bin
    ├───contrib
    ├───doc
    ├───dvi
    ├───etc
    ├───help
    ├───html
    ├───include
    ├───info
    ├───lib
    ├───libexec
    ├───man
    ├───manifest
    ├───misc
    ├───pdf
    ├───ps
    ├───sbin
    ├───share
    ├───Start Menu
    └───var
```

Setting up environment variables to use tools.
Recommended to create batch file to set environment variables for development work only (since some gnuwin32 tools conflict with built-in windows ones).

Example batch file

```cmd
SET TOOLS_DIR=C:\DEV

SET PATH=%PATH%;%TOOLS_DIR%\avr8-gnu-toolchain\bin
SET PATH=%PATH%;%TOOLS_DIR%\avr8-gnu-toolchain\avr\bin
SET PATH=%PATH%;%TOOLS_DIR%\gnuwin32\bin
SET PATH=%PATH%;%TOOLS_DIR%\avrdude-6.4-mingw32

start cmd
```

Testing


```
avr-gcc main.c -o main.elf
avr-objcopy main.elf -O ihex main.hex
avrdude -c usbasp -p t85 -U flash:w:"main.hex":a
```







## In Linux

Steps:
1. Install AVR toolchain (to build the code)
2. Install GnuWin32 (for supporting build tools like Make)
3. Install AVR Dude (to flash the chip)
4. Install VSCode (as an IDE) [Optional]


```bash
apt install avrdude
apt install gcc-avr?

#make is usually already installed, but if not...
#apt install make

# Another option...
#apt install arduino

#in WSL then install usbip to access USB devices in the host
#sudo apt install linux-tools-virtual hwdata
#sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*/usbip 20
```



Using Arduino-cli...

You can using the board details command to figure out the fully qualified board name you want...
e.g. 
```
arduino-cli board listall
```
shows
```
Board Name                       FQBN
ATtiny24/44/84                   attiny:avr:ATtinyX4
ATtiny25/45/85                   attiny:avr:ATtinyX5
Adafruit Circuit Playground      arduino:avr:circuitplay32u4cat
Arduino BT                       arduino:avr:bt
Arduino Duemilanove or Diecimila arduino:avr:diecimila
Arduino Esplora                  arduino:avr:esplora
Arduino Ethernet                 arduino:avr:ethernet
Arduino Fio                      arduino:avr:fio
Arduino Gemma                    arduino:avr:gemma
Arduino Industrial 101           arduino:avr:chiwawa
Arduino Leonardo                 arduino:avr:leonardo
Arduino Leonardo ETH             arduino:avr:leonardoeth
Arduino Mega ADK                 arduino:avr:megaADK
Arduino Mega or Mega 2560        arduino:avr:mega
Arduino Micro                    arduino:avr:micro
Arduino Mini                     arduino:avr:mini
Arduino NG or older              arduino:avr:atmegang
Arduino Nano                     arduino:avr:nano
Arduino Pro or Pro Mini          arduino:avr:pro
Arduino Robot Control            arduino:avr:robotControl
Arduino Robot Motor              arduino:avr:robotMotor
Arduino Uno                      arduino:avr:uno
Arduino Uno WiFi                 arduino:avr:unowifi
Arduino Yún                      arduino:avr:yun
Arduino Yún Mini                 arduino:avr:yunmini
LilyPad Arduino                  arduino:avr:lilypad
LilyPad Arduino USB              arduino:avr:LilyPadUSB
Linino One                       arduino:avr:one
```

I know I want ATtiny85... but the board option `attiny:avr:ATtinyX5` isn't specific enough.  So let's check the board details:

```
arduino-cli board details attiny:avr:ATtinyX5
```
shows
```
Board name:            ATtiny25/45/85
FQBN:                  attiny:avr:ATtinyX5
Board version:         1.0.2

Package name:          attiny
Package maintainer:    David A. Mellis
Package URL:           https://raw.githubusercontent.com/damellis/attiny/ide-1.6.x-boards-manager/package_damellis_attiny_index.json
Package website:       https://github.com/damellis/attiny

Platform name:         attiny
Platform category:     attiny
Platform architecture: avr
Platform URL:          https://github.com/damellis/attiny/archive/6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform file name:    6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform size (bytes): 5913
Platform checksum:     SHA-256:1654a8968fea7d599255bd18430786f9c84340606e7a678b9cf9a3cd49d94ad7

Option:                Processor                                                                                                cpu
                       ATtiny25                                                                                            ✔    cpu=attiny25
                       ATtiny45                                                                                                 cpu=attiny45
                       ATtiny85                                                                                                 cpu=attiny85
Option:                Clock                                                                                                    clock
                       Internal 1 MHz                                                                                      ✔    clock=internal1
                       Internal 8 MHz                                                                                           clock=internal8
                       Internal 16 MHz                                                                                          clock=internal16
                       External 8 MHz                                                                                           clock=external8
                       External 16 MHz                                                                                          clock=external16
                       External 20 MHz                                                                                          clock=external20
Programmers:           Id                                                                                                  Name
```

Notice the cpu and clock options? Notice has ATtiny25 and internal 1 MHz are selected?  Great if you want that, but what if you don't?  You can add addition options and use the same board details command to confirm you've got it right...


```
arduino-cli board details attiny:avr:ATtinyX5:cpu=attiny85,clock=internal8
```
shows
```
Board name:            ATtiny25/45/85
FQBN:                  attiny:avr:ATtinyX5
Board version:         1.0.2

Package name:          attiny
Package maintainer:    David A. Mellis
Package URL:           https://raw.githubusercontent.com/damellis/attiny/ide-1.6.x-boards-manager/package_damellis_attiny_index.json
Package website:       https://github.com/damellis/attiny

Platform name:         attiny
Platform category:     attiny
Platform architecture: avr
Platform URL:          https://github.com/damellis/attiny/archive/6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform file name:    6bba7d452af59d5190025bc870ec9e53d170e4d9.zip
Platform size (bytes): 5913
Platform checksum:     SHA-256:1654a8968fea7d599255bd18430786f9c84340606e7a678b9cf9a3cd49d94ad7

Option:                Processor                                                                                                cpu
                       ATtiny25                                                                                                 cpu=attiny25
                       ATtiny45                                                                                                 cpu=attiny45
                       ATtiny85                                                                                            ✔    cpu=attiny85
Option:                Clock                                                                                                    clock
                       Internal 1 MHz                                                                                           clock=internal1
                       Internal 8 MHz                                                                                      ✔    clock=internal8
                       Internal 16 MHz                                                                                          clock=internal16
                       External 8 MHz                                                                                           clock=external8
                       External 16 MHz                                                                                          clock=external16
                       External 20 MHz                                                                                          clock=external20
Programmers:           Id                                                                                                  Name
```


