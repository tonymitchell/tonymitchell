---
title: How to setup the AVR toolchain on Linux
date: 2022-05-21 10:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, avrdude, avr-gcc]
---

> ## Tutorial Versions
> 
> I have written 3 different versions of this tutorial that cover the following different setups:
> 1. [Windows-based AVR toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %})
> 
>    This options primarily uses Windows-based tools and has two variations depending on how you run make
>    - Windows + GnuWin32
>    - Windows + WSL
>
> 2. [WSL-based AVR toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})
>
>    This options primarily uses WSL-based linux tools and has two variations depending on how you program the chip -- either WSL-based avrdude using usbip to access the USB programmer or Windows-based avrdude to natively access the USB programmer
>    - WSL + usbip
>    - WSL + Windows
>
> 3. **Linux-based AVR toolchain** (*this guide*)
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

This guide will help you get your environment set up to build projects on the Atmel AVR chips (e.g. ATmega328, ATtiny85, etc.) projects on Windows using **Linux-based** (Ubuntu) tools.

Summary:
1. [Option 1: The Quick & Easy Way](#option-1-the-quick--easy-way) (Recommended)
1. [Option 2: The Latest & Manual Way](#option-2-the-latest--manual-way)
    1. [Install AVR toolchain](#install-avr-toolchain) (to build the code)
    1. [Install AVR Dude](#install-avr-dude) (to flash the chip)
    1. [Configure PATH](#configure-path) (to make tools available)
1. [Testing the AVR toolchain](#testing-the-avr-toolchain)
1. [FAQ](#faq)


## Option 1: The Quick & Easy Way

All the packages we need are already available in Ubuntu. Depending on your distribution release, the tools may not be the latest available.  For example, only version 6.3 of avrdude is available while versions 6.4 and 7.0 can be installed manually.  avr-gcc is better, with Ubuntu 22.04 including the latest release of 3.6.2.

> |Tool|20.04 Focal (LTS)|22.04 Jammy (LTS)|
> |-|:-:|:-:|
> |avr-gcc|[3.6.1](# "1:5.4.0+Atmel3.6.1-2build1")  |[3.6.2](# "1:5.4.0+Atmel3.6.2-3")|
> |avrdude|[6.3](# "6.3-20171130+svn1429-2")        |[6.3](# "6.3-20171130+svn1429-2")|

We can install all the tools we need in a single command.

```bash
$ sudo apt install make gcc-avr avr-libc avrdude
```

Skip ahead to [Testing the AVR toolchain](#testing-the-avr-toolchain)


## Option 2: The Latest & Manual Way

This option will enable you to install the latest version of each of the tools, but requires manual installation. I don't recommend this approach, but if you're experiencing an issue with an older version and need the latest version you can follow some or all of these steps to install the latest version.  NOTE: If you're on a recent version of Ubuntu, avr-gcc tends to be up-to-date. It's only avrdude that doesn't include the latest version.

### Prep

In this option we will manually install the latest version of the tools into your home directory under `~/.local/`.  You may need to create it if it doesn't already exist.

```bash
$ mkdir ~/.local
```

### Install AVR toolchain

#### Download

The first step is to download the AVR toolchain from the Microchip website. The toolchain can be downloaded as part of the Atmel Studio IDE or standalone.  For this guide we will use the standalone toolchain.

1. Browse to toolchain download page at: 
- [https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers)

2. From the downloads list, find and download the appropriate toolchain for your chip and dev OS. For this guide we will download the toolchain for **AVR 8-bit** chips (like ATmega328 or ATtiny) that runs on **Linux 64-bit** since WSL is a 64-bit Linux. At the time of writing that was [AVR 8-bit Toolchain 3.6.2 - Linux 64-bit](https://www.microchip.com/mymicrochip/filehandler.aspx?ddocname=en607660). NOTE: You will have to sign up for a free account to complete the download
- At the time of writing the main download page wasn't working, so alternatively you can download the prior version ([AVR 8-bit Toolchain 3.6.1 - Linux 64-bit](https://ww1.microchip.com/downloads/Secure/en/DeviceDoc/avr8-gnu-toolchain-3.6.1.1752-linux.any.x86_64.tar.gz)) release from the archives at: [https://www.microchip.com/en-us/tools-resources/archives/avr-sam-mcus#AVR and Arm-Based MCU Toolchains](https://www.microchip.com/en-us/tools-resources/archives/avr-sam-mcus#AVR%20and%20Arm-Based%20MCU%20Toolchains)

3. I also recommend downloading the Release Notes for your toolchain (if any).  They should be available on the same downloads page. (e.g. [AVR 8-bit Toolchain 3.6.1 - Release Note](https://ww1.microchip.com/downloads/Secure/en/DeviceDoc/avr8-gnu-toolchain-3.6.1.1752-readme.pdf)).  The release notes can contain important installation instructions (like the fact that you may need to download chip packs)

4. **[Optional]** IF the chip you are using isn't supported out-of-the-box by the main toolchain, browse to: http://packs.download.atmel.com/ to find and download the appropriate chip packs. Note: most of the popular chips should already be supported).
    - Even if you don't NEED the chip packs, there is additional documentation in them that you might find valuable to read.
    - The chip pack files end in .atpack, but are actually .zip archive files and needs to be expanded to be used.
    - If you need to use a chip pack, you will need to reference the directory via -B and -I options when compiling. For example:  `avr-gcc -mmcu=atmega328pb -B ~/avr-device-packs/Atmel.ATmega_DFP.1.0.86/gcc/dev/atmega328pb/ -I ~/avr-device-packs/Atmel.ATmega_DFP.1.0.86/include/ `

#### Install

Unzip the downloaded toolchain archive (e.g. avr8-gnu-toolchain-3.6.1.1752-linux.any.x86_64.tar.gz) in your home directory. You should now have a directory named avr8-gnu-toolchain-linux_x86_64 in your home directory. The contents of this folder need to be copied to your ~/.local/ folder.

```bash
cd
tar xvzf avr8-gnu-toolchain-3.6.1.1752-linux.any.x86_64.tar.gz
cp -a ~/avr8-gnu-toolchain-linux_x86_64/* ~/.local/
```

Our additions to the home directory so far:
```
~/
└───.local
    ├───avr
    ├───bin
    ├───doc
    └───...
```

**[Optional]** If you choose to download the chi ppacks as well, I recommend unzipping those into a separate directory for chip packs (e.g. ~/avr-device-packs) with each chip pack in a separate subfolder (~/avr-device-packs/Atmel.ATmega_DFP.2.0.401/).

Our additions to the home directory so far:
```
~/
├───avr-device-packs
│   ├───Atmel.ATmega_DFP.2.0.401
│   │   ├───gcc
│   │   ├───include
│   │   └───...
│   └───Atmel.ATtiny_DFP.2.0.368
└───.local
```

#### Test
To test the installation of the avr-gcc toolchain, run `avr-gcc --version` in the `~/.local/bin` directory. The output should look something this:

```
~/.local/bin$ ./avr-gcc --version
avr-gcc (AVR_8_bit_GNU_Toolchain_3.6.2_1778) 5.4.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

### Install AVR Dude

#### Download

1. Browse to the AVR Dude releases directory on GitHub: https://github.com/avrdudes/avrdude/releases
2. Download the latest version for your OS (e.g. [avrdude-7.0.tar.gz](https://github.com/avrdudes/avrdude/releases/download/v7.0/avrdude-7.0.tar.gz)) to your home directory.

> If you're looking for older versions such as v6.4 or prior, they are can be found at their old site: http://download.savannah.gnu.org/releases/avrdude/ (e.g [avrdude-6.4.tar.gz](http://download.savannah.gnu.org/releases/avrdude/avrdude-6.4.tar.gz))

```bash
# Download release
wget https://github.com/avrdudes/avrdude/releases/download/v7.0/avrdude-7.0.tar.gz
```

#### Install

This will install avrdude into your $HOME/.local structure.  You can update the configure command if you wish to install it elsewhere.

```bash
# Expand archive
tar xvzf avrdude-7.0.tar.gz
cd avrdude-7.0/

# Build avrdude (See https://www.nongnu.org/avrdude/user-manual/avrdude_18.html#Unix)
./configure --prefix=$HOME/.local
make
make install
```


#### Test

To test the installation of avrdude, run `avrdude` in the `.local/bin` directory. The output should look something this:

```
~/.local/bin$ ./avrdude
Usage: avrdude [options]
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



### Configure PATH

At this point all the tools are installed, but we still need to make them available on the PATH. Since some of the GnuWin32 tools can conflict with built-in Windows tools of the same name, we'll only add these tools to the PATH when we need them via a batch file.  

Here is a summary of what you've done so far:

|Zip|Target Location|Install|
|-|-|-|
|avr8-gnu-toolchain-3.6.2.1778-win32.any.x86.zip|~/.local/|Unzip to target|
|avrdude-v7.0-windows-x86.zip                   |~/.local/|Build and install to target|
|Chip packs (*.atpack)                          |~/avr-device-packs/|Unzip to target|


In many distributions of linux, including the WSL default Ubuntu, the .local/bin directory will be added to your path automatically and we won't have to do anything else except log out and log back in.  Log out and back in, and then try running `avr-gcc --version` and `avrdude -v` to see if they're now in your path.

If so, you're all done.  If not, edit your .profile and add that block. 

Take a look at your ~/.profile and see if there is block like:
```bash
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
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

In the directory you created the file, run the following commands. Make sure you plug in your Arduino before running avrdude and update the device port to match the one assigned to your Arduino.
```
avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
avr-objcopy blink.elf -O ihex blink.hex
sudo avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P /dev/ttyACM0
```

If everything is working you should see something like.
```console
$ avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
$ avr-objcopy blink.elf -O ihex blink.hex
$ sudo avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P /dev/ttyACM0

avrdude.exe: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.00s

avrdude.exe: Device signature = 0x1e950f (probably m328p)
avrdude.exe: NOTE: "flash" memory has been specified, an erase cycle will be performed
             To disable this feature, specify the -D option.
avrdude.exe: erasing chip
avrdude.exe: reading input file "blink.hex"
avrdude.exe: input file blink.hex auto detected as Intel Hex
avrdude.exe: writing flash (176 bytes):

Writing | ################################################## | 100% 0.04s

avrdude.exe: 176 bytes of flash written
avrdude.exe: verifying flash memory against blink.hex:
avrdude.exe: input file blink.hex auto detected as Intel Hex

Reading | ################################################## | 100% 0.03s

avrdude.exe: 176 bytes of flash verified

avrdude.exe done.  Thank you.
```


## FAQ

How do I find out what port my Arduino is using?
: TBD - Easiest is dmesg / lsusb

How do I run avrdude without sudo?
: TBD - udev rule
