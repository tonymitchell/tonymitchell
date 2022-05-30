---
title: How to setup the AVR toolchain on Linux
date: 2022-05-21 10:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, avrdude, avr-gcc]
---

## Overview

This guide will help you get your environment set up to build projects on the Atmel AVR chips (e.g. ATmega328, ATtiny85, etc.) projects on Windows using **Linux-based** (Ubuntu) tools.

Summary:
1. [Install the AVR tools](#install-the-avr-tools)
1. [Testing the AVR toolchain](#testing-the-avr-toolchain)
1. [Next Steps](#next-steps)
1. [Troubleshooting](#troubleshooting)


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
{: .prompt-info }


## Install the AVR tools

All the packages we need are already available in Ubuntu. Depending on your distribution release, the tools may not be the latest, but should be fine in most cases.  

>If you need the latest versions or need to do a manual install for some reason, follow this guide instead: [How to manually install AVR tools on Linux (or WSL2)]({% post_url 2022-05-21-manually-install-avr-tools-on-linux %})
{: .prompt-tip }

> |Tool|20.04 Focal (LTS)|22.04 Jammy (LTS)|
> |-|:-:|:-:|
> |avr-gcc|[3.6.1](https://packages.ubuntu.com/focal/gcc-avr "1:5.4.0+Atmel3.6.1-2build1"){:target="_blank"}  |[3.6.2](https://packages.ubuntu.com/jammy/gcc-avr "1:5.4.0+Atmel3.6.2-3"){:target="_blank"}|
> |avrdude|[6.3](https://packages.ubuntu.com/focal/avrdude "6.3-20171130+svn1429-2"){:target="_blank"}        |[6.3](https://packages.ubuntu.com/jammy/avrdude "6.3-20171130+svn1429-2"){:target="_blank"}|

We can install all the tools we need in a single command.

```bash
$ sudo apt install make gcc-avr avr-libc avrdude
```

Let's run a quick check that all the tools are installed before we continue by running following commands:
```bash
make -v
avr-gcc --version
avrdude -v
```

The output should look something like this:
```console
$ make -v
GNU Make 4.2.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
$ avr-gcc --version
avr-gcc (GCC) 5.4.0
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$  avrdude -v

avrdude: Version 6.3-20171130
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2014 Joerg Wunsch

         System wide configuration file is "/etc/avrdude.conf"
         User configuration file is "/home/tony/.avrduderc"
         User configuration file does not exist or is not a regular file, skipping


avrdude: no programmer has been specified on the command line or the config file
         Specify a programmer using the -c option and try again
```

If any of those produced an error or could not be found, you'll need to re-visit the installation steps before continuing.


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
{: file="blink.c" }

In the directory you created the file, run the following commands. Make sure you plug in your Arduino before running avrdude and update the device port to match the one assigned to your Arduino.
```
avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
avr-objcopy blink.elf -O ihex blink.hex
avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P /dev/ttyACM0
```

If everything is working you should see something like.
```console
$ avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
$ avr-objcopy blink.elf -O ihex blink.hex
$ avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P /dev/ttyACM0

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

## Next Steps

That's all there is to it.  To continue setting up your development environment check out my other posts on building projects with a Makefile and configuring VS Code:
- [Building AVR projects with make](#)
- [Use VS Code with the AVR toolchain](#)

## Troubleshooting

How do I find out what port my Arduino is using?
: TBD - Easiest is dmesg / lsusb

How do I run avrdude without sudo?
: TBD - udev rule / plugdev group

