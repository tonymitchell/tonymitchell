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
> 3. [Linux-based]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})
>
>    This option uses all native linux-based tools on a native Linux installation.
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
1. [Prep](#prep)
1. [Install AVR toolchain](#install-avr-toolchain) (to build the code)
1. [Install AVR Dude](#install-avr-dude) (to flash the chip)
1. [Install GnuWin32](#install-gnuwin32) (for supporting build tools like Make)
1. [Configure PATH](#configure-path) (to make tools available)
1. [Testing the AVR toolchain](#testing-the-avr-toolchain)
1. [FAQ](#faq)

<!-- 
1. [Install VS Code](#install-vs-code) (as an IDE) [Optional]
-->

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


Tools directory so far:
```
C:\AVR
└───avr8-gnu-toolchain
    ├───avr
    ├───bin
    ├───doc
    └───...
```

**[Optional]** If you choose to download the chi ppacks as well, I recommend unzipping those into a separate directory for chip packs with each chip pack in a separate subfolder.

Tools directory so far:
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

### Test
To test the installation of the avr-gcc toolchain, run `avr-gcc --version` in the `avr8-gnu-toolchain\bin` directory. The output should look something this:

```
C:\AVR\avr8-gnu-toolchain\bin>avr-gcc --version
avr-gcc (AVR_8_bit_GNU_Toolchain_3.6.2_1778) 5.4.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

## Install AVR Dude

### Download

1. Browse to the AVR Dude releases directory on GitHub: https://github.com/avrdudes/avrdude/releases
2. Download the latest version for your OS (e.g. [avrdude-v7.0-windows-x86.zip](https://github.com/avrdudes/avrdude/releases/download/v7.0/avrdude-v7.0-windows-x86.zip)).

> If you're looking for older versions such as v6.4 or prior, they are can be found at their old site: http://download.savannah.gnu.org/releases/avrdude/ (e.g [avrdude-6.4-mingw32.zip](http://download.savannah.gnu.org/releases/avrdude/avrdude-6.4-mingw32.zip))

### Install

Unzip the downloaded archive (e.g. avrdude-v7.0-windows-x86.zip) into the tools folder into a directory named "avrdude". This will keep it version and platform agnostic.

Tools directory so far:
```
C:\AVR
├───avr8-gnu-toolchain
└───avrdude
    ├───avrdude.exe
    └───... 
```

### Test

To test the installation of avrdude, run `avrdude` in the `avrdude` directory. The output should look something this:

```
C:\DEV\AVR\avrdude>avrdude.exe
Usage: avrdude.exe [options]
Options:
  -p <partno>                Required. Specify AVR device.
  -b <baudrate>              Override RS-232 baud rate.
  -B <bitclock>              Specify JTAG/STK500v2 bit clock period (us).
  -C <config-file>           Specify location of configuration file.
  -c <programmer>            Specify programmer type.
  -D                         Disable auto erase for flash memory
  -i <delay>                 ISP Clock Delay [in microseconds]
  -P <port>                  Specify connection port.
  -F                         Override invalid signature check.
  -e                         Perform a chip erase.
  -O                         Perform RC oscillator calibration (see AVR053).
  -U <memtype>:r|w|v:<filename>[:format]
                             Memory operation specification.
                             Multiple -U options are allowed, each request
                             is performed in the order specified.
  -n                         Do not write anything to the device.
  -V                         Do not verify.
  -t                         Enter terminal mode.
  -E <exitspec>[,<exitspec>] List programmer exit specifications.
  -x <extended_param>        Pass <extended_param> to programmer.
  -v                         Verbose output. -v -v for more.
  -q                         Quell progress output. -q -q for less.
  -l logfile                 Use logfile rather than stderr for diagnostics.
  -?                         Display this usage.

avrdude version 7.0, URL: <https://github.com/avrdudes/avrdude>
```


## Install GnuWin32

The [GnuWin32 project](http://gnuwin32.sourceforge.net/) provides ports of GNU tools to Windows.  For our purposes, the main tool we'll need from GnuWin32 is `make`. You can download and install just `make`, but my preferences is to download and install the full tool suite because there are lots of additional useful tools in there, and that's what we'll do in this guide.

### Download

1. Browse to [https://sourceforge.net/projects/getgnuwin32/files/getgnuwin32/0.6.30/](https://sourceforge.net/projects/getgnuwin32/files/getgnuwin32/0.6.30/) and download [GetGnuWin32-0.6.3.exe](https://sourceforge.net/projects/getgnuwin32/files/getgnuwin32/0.6.30/GetGnuWin32-0.6.3.exe/download).
    - If for some reason that link doesn't work, you can start back on the [project site](http://getgnuwin32.sourceforge.net/) and navigate to the downloads folder from there.
2. Save that file to a temporary location such as C:\temp or your Downloads folder.

### Install

1. Run the executable you downloaded (e.g GetGnuWin32-0.6.3.exe). 
1. When presented, **Accept** the licence. 
1. You will be presented with a summary of installation instructions which should align to the steps below.
1. Click the **Install** button. This will create a new folder in the same location the executable was located (e.g. C:\temp)
1. Open a Command Prompt as Administrator to ensure the next steps have permission necessary to install.
1. Change directory to the new folder created when you ran the installer. (e.g. C:\temp\GetGnuWin32)
    ```
    C:\> cd \temp\GetGnuWin32
    ```
1. Run download.bat.  This will download all the packages necessary for gnuwin32. You will be prompted to press a key a few times before downloads start. The downloads can take a while even with a fast internet connection.
    ```
    C:\temp\GetGnuWin32> download.bat
    ```
1. If there any failed downloads, run the download.bat again until they are all successfull.
1. Once all the packages have been downloaded successfully, run the install.bat and pass it the target directory where gnuwin32 should be installed.  If your tools directory is C:\AVR, then you would pass C:\AVR\gnuwin32 as in the example below.
    ```
    C:\temp\GetGnuWin32> install.bat C:\AVR\gnuwin32
    ```

1. You will be prompted about installing updated versions of several components. When prompted, just hit Enter to accept the default responses to install these updated components.  

Tools directory so far:
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

### Test

To test the installation of gnuwin32, run `make -v` in the `gnuwin32\bin` directory. The output should look something this:

```
C:\DEV\gnuwin32\bin\>make -v
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i386-pc-mingw32
```

## Configure PATH

At this point all the tools are installed, but we still need to make them available on the PATH. Since some of the GnuWin32 tools can conflict with built-in Windows tools of the same name, we'll only add these tools to the PATH when we need them via a batch file.  

Here is a summary of what you've done so far:

|Zip|Target Location|Install|
|-|-|-|
|avr8-gnu-toolchain-3.6.2.1778-win32.any.x86.zip|C:\AVR\avr8-gnu-toolchain|Unzip to target|
|avrdude-v7.0-windows-x86.zip                   |C:\AVR\avrdude|Unzip to target|
|GetGnuWin32-0.6.3.exe                          |C:\AVR\gnuwin32|Installer, download.bat, install.bat C:\AVR\gnuwin32|
|Chip packs (*.atpack)                          |C:\AVR\avr8-dvp|Unzip to target|

We'll create two separate batch files -- one for setting the paths, and another for opening a command prompt with the paths set. This way if you want to just add the paths to an existing console session you can run the first, but if you want a brand new session you can run the second.

Create a new file in your tools directory named `set_avr_paths.cmd`, and paste in the following:
```batchfile
@ECHO OFF
ECHO Setting AVR tool paths
SET TOOLS_DIR=C:\AVR

SET PATH=%PATH%;%TOOLS_DIR%\avr8-gnu-toolchain\bin
SET PATH=%PATH%;%TOOLS_DIR%\avrdude
SET PATH=%PATH%;%TOOLS_DIR%\gnuwin32\bin
```

Create another new file in your tools directory named `avr_dev_prompt.cmd`, and paste in the following:

```batchfile
@TITLE AVR Dev Command Prompt
@%comspec% /k " cd \ & "%~dp0set_avr_paths.cmd" "
```


## Testing the AVR toolchain

To test that everything is working correctly, let's try building a sample program and writing it to an Arduino.  If you don't have an Arduino handy, you can either skip the `avrdude` step or adjust the program and commands to fit your scenario.

In a new directory, create a new file named blink.c and paste the following program into it.  This is a AVR implementation of the classic Arduino Blink sketch.  If you load this to an Arduino it will make the built-in LED (pin 13/LED_BUILTIN) flash on and off every 500ms.

```c
#include <avr/io.h>
#include <util/delay.h>

int main()
{
    // Set built-in LED pin as output
    DDRB |= (1 << DDB5);
    while (1) {
        PORTB |=  (1 << PB5);   // LED on
        _delay_ms(500);
        PORTB &= ~(1 << PB5);   // LED off
        _delay_ms(500);
    }
    return 0;
}
```

Double-click the `avr_dev_prompt.cmd` batch file you created above to prepare a console session with tools on the PATH. 

In the directory you created the file, run the following commands. Make sure you plug in your Arduino before running avrdude and update the COM port to match the one assigned to your Arduino.
```
avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
avr-objcopy blink.elf -O ihex blink.hex
avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P COM4
```

If everything is working you should see something like.
```
C:\temp\blink>avr-gcc blink.c -o blink.elf -mmcu=atmega328p -DF_CPU=16000000UL -Os

C:\temp\blink>avr-objcopy blink.elf -O ihex blink.hex

C:\temp\blink>avrdude -c arduino -p atmega328p -U flash:w:"blink.hex":a -P COM4

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.00s

avrdude: Device signature = 0x1e950f (probably m328p)
avrdude: NOTE: "flash" memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "blink.hex"
avrdude: input file blink.hex auto detected as Intel Hex
avrdude: writing flash (176 bytes):

Writing | ################################################## | 100% 0.04s

avrdude: 176 bytes of flash written
avrdude: verifying flash memory against blink.hex:
avrdude: input file blink.hex auto detected as Intel Hex

Reading | ################################################## | 100% 0.03s

avrdude: 176 bytes of flash verified

avrdude done.  Thank you.
```


## FAQ

How do I find out what COM port my Arduino is using?
: Open **Device Manager**. Expand **Ports (COM & LPT)**. You should see an entry named something like  "Arduino Uno (COM4)".  The COM port is in parentheses.


<!--
## Install VS Code

-->