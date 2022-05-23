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
>
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

Summary:
1. Prep
1. Install AVR toolchain (to build the code)
1. Install AVR Dude (to flash the chip)
1. Install GnuWin32 (for supporting build tools like Make)
1. Install VSCode (as an IDE) [Optional]


## Prep

Start by creating a folder to hold all your dev tools (e.g. C:\Tools\AVR or C:\Users\<user>\AVR).  I will be using C:\AVR in this guide.

## Install AVR toolchain

### Download

The first step is to download the AVR toolchain from the Microchip website. The toolchain can be downloaded as part of the Atmel Studio IDE or standalone.  For this guide we will use the standalone toolchain and use VS Code as the IDE.

1. Browse to toolchain download page at: 
- [https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers)

2. From the downloads list, find and download the appropriate toolchain for your chip and dev OS. For this guide we will download the toolchain for **AVR 8-bit** chips (like ATmega328 or ATtiny) that runs on **Windows**. At the time of writing that was [AVR 8-bit Toolchain v3.62 – Windows](https://www.microchip.com/mymicrochip/filehandler.aspx?ddocname=en607654). NOTE: You will have to sign up for a free account to complete the download
- At the time of writing the main download page wasn't working, so alternatively you can download the prior version ([3.6.1](https://ww1.microchip.com/downloads/Secure/en/DeviceDoc/avr8-gnu-toolchain-3.6.1.1752-win32.any.x86.zip)) release from the archives at: [https://www.microchip.com/en-us/tools-resources/archives/avr-sam-mcus#AVR and Arm-Based MCU Toolchains](https://www.microchip.com/en-us/tools-resources/archives/avr-sam-mcus#AVR%20and%20Arm-Based%20MCU%20Toolchains)

3. I also recommend downloading the Release Notes for your toolchain (if any).  They should be available on the same downloads page. (e.g. [AVR 8-bit Toolchain 3.6.1 - Release Note](https://ww1.microchip.com/downloads/Secure/en/DeviceDoc/avr8-gnu-toolchain-3.6.1.1752-readme.pdf)).  The release notes can contain important installation instructions (like the fact that you may need to download chip packs)

4. **[Optional]** IF the chip you are using isn't supported out-of-the-box by the main toolchain, browse to: http://packs.download.atmel.com/ to find and download the appropriate chip packs. Note: most of the popular chips should already be supported).
    - Even if you don't NEED the chip packs, there is additional documentation in them that you might find valuable to read.
    - The chip pack files end in .atpack, but are actually .zip archive files and needs to be expanded to be used.
    - If you need to use a chip pack, you will need to reference the directory via -B and -I options when compiling. For example:  `avr-gcc -mmcu=atmega328pb -B /home/packs/Atmel.ATmega_DFP.1.0.86/gcc/dev/atmega328pb/ -I /home/packs/Atmel.ATmega_DFP.1.0.86/include/ `

### Install

Unzip the downloaded toolchain archive (e.g. avr8-gnu-toolchain-3.6.1.1752-win32.any.x86.zip) into the tools folder you created earlier. You should now have a directory named avr8-gnu-toolchain-win32_x86 in your tools folder. My preference is to rename the directory to remove any platform specific names (e.g. win32_x86) to make any scripts I create platform agnostic and more portable (e.g. avr8-gnu-toolchain).


Tools folder so far:
```
C:\AVR
└───avr8-gnu-toolchain
    ├───avr
    ├───bin
    ├───doc
    └───...
```

**[Optional]** If you choose to download the chi ppacks as well, I recommend unzipping those into a separate directory for chip packs with each chip pack in a separate subfolder.

```
C:\AVR
├───avr-device-packs
│   ├───Atmel.ATmega_DFP.2.0.401
│   │   ├───gcc
│   │   ├───include
│   │   └───...
│   └───Atmel.ATtiny_DFP.2.0.368
└───avr8-gnu-toolchain
```


## Install AVR Dude

### Download

1. Browse to the AVR Dude releases directory on GitHub: https://github.com/avrdudes/avrdude/releases
2. Download the latest version for your OS (e.g. [avrdude-v7.0-windows-x86.zip](https://github.com/avrdudes/avrdude/releases/download/v7.0/avrdude-v7.0-windows-x86.zip)).

> If you're looking for older versions such as v6.4 or prior, they are can be found at their old site: http://download.savannah.gnu.org/releases/avrdude/ (e.g [avrdude-6.4-mingw32.zip](http://download.savannah.gnu.org/releases/avrdude/avrdude-6.4-mingw32.zip))

### Install

Unzip the downloaded archive (e.g. avrdude-v7.0-windows-x86.zip) into the tools folder into a directory named "avrdude". This will keep it version and platform agnostic.

```
C:\AVR
├───avr8-gnu-toolchain
└───avrdude
    ├───avrdude.exe
    └───... 
```

## Install GnuWin32

The [GnuWin32 project](http://gnuwin32.sourceforge.net/) provides ports GNU tools to Windows.  For our purposes, the main tool we'll need is `make`. You can download and install just make, but my preferences is to download and install the full tool suite, and that's what we'll do in this guide.

### Download

1. Browse to https://sourceforge.net/projects/getgnuwin32/files/getgnuwin32/0.6.30/ and download GetGnuWin32-0.6.3.exe.  Direct link here: https://sourceforge.net/projects/getgnuwin32/files/getgnuwin32/0.6.30/GetGnuWin32-0.6.3.exe/download
    - If for some reason that link doesn't work, you can start back on the project site by browsing to http://getgnuwin32.sourceforge.net/ and navigate to the downloads folder from there.
2. Save that file to a temporary location such as C:\temp or your Downloads folder.

### Install

1. Run the executable you downloaded (e.g GetGnuWin32-0.6.3.exe). 
1. When presented, accept the licence. 
1. You will be presented with a summary of installation instructions which should align to the steps below.
1. Click the Install button. This will create a new folder in the same location the executable was located (e.g. C:\temp)
1. Open a Command Prompt as Administrator to ensure the next steps have permission necessary to install.
1. Change directory to the new folder created when you ran the installer. (e.g. C:\temp\GetGnuWin32)
    ```cmd
    > cd \temp\GetGnuWin32
    ```
1. Run download.bat.  This will download all the packages necessary for gnuwin32. You will be prompted to press a key a few times before downloads start. The downloads can take a while even with a fast internet connection.
    ```cmd
    > download.bat
    ```
1. If there any failed downloads, run the download.bat again until they are all successfull.
1. Once all the packages have been downloaded successfully, run the install.bat and pass it the target directory where gnuwin32 should be installed.  If your tools directory is C:\AVR, then you would pass C:\AVR\gnuwin32 as in the example below.
    ```cmd
    > install.bat C:\AVR\gnuwin32
    ```

```
C:\AVR
├───avr8-gnu-toolchain
├───avrdude
└───gnuwin32
    ├───bin
    ├───contrib
    ├───doc
    └───... 
```

## Configuring

Here is a summary of what you've done so far:

|Zip|Target Location|Install|
|-|-|-|
|avr8-gnu-toolchain-3.6.2.1778-win32.any.x86.zip|C:\AVR\avr8-gnu-toolchain|Unzip to target|
|avrdude-v7.0-windows-x86.zip                   |C:\AVR\avrdude|Unzip to target|
|GetGnuWin32-0.6.3.exe                          |C:\AVR\gnuwin32|Run installer, and complete steps in Readme.txt to produce gnuwin32 folder. At a high-level, those steps are: Run download.bat, the run install.bat, then copy folder, then run update-links.bat in the target.|
|Chip packs (*.atpack)                          |C:\AVR\avr8-dvp|Unzip to target|



The first few levels of the target structure should look like (note: the avr8-dvp is optional and in this example has several chip packs installed):
```




C:\AVR
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
├───avrdude
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
SET TOOLS_DIR=C:\AVR

SET PATH=%PATH%;%TOOLS_DIR%\avr8-gnu-toolchain\bin
SET PATH=%PATH%;%TOOLS_DIR%\avr8-gnu-toolchain\avr\bin
SET PATH=%PATH%;%TOOLS_DIR%\avrdude
SET PATH=%PATH%;%TOOLS_DIR%\gnuwin32\bin

start cmd
```

## Testing the AVR toolchain


```
avr-gcc main.c -o main.elf
avr-objcopy main.elf -O ihex main.hex
avrdude -c usbasp -p t85 -U flash:w:"main.hex":a
```


