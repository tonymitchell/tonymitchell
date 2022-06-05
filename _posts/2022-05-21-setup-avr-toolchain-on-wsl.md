---
title: How to setup the AVR toolchain on WSL2 (Windows 10+)
date: 2022-05-21 10:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, avrdude, avr-gcc]
---

## Overview

This guide will help you get your environment set up to build projects on the Atmel AVR chips (e.g. ATmega328, ATtiny85, etc.) projects on Windows using **WSL-based** tools. 

Summary:
1. [Install the AVR tools](#install-the-avr-tools)
1. [Testing the AVR toolchain](#testing-the-avr-toolchain)
1. [Programming the chip](#programming-the-chip)
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
> 2. **WSL-based toolchain** (*this guide*)
>
>    This options primarily uses WSL-based linux tools and has two variations depending on how you program the chip -- either WSL-based avrdude using usbip to access the USB programmer or Windows-based avrdude to natively access the USB programmer
>    - WSL + usbip
>    - WSL + Windows
>
> 3. [Linux-based toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})
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

In the directory you created the file, run the following commands. Make sure you plug in your Arduino before running avrdude and update the COM port to match the one assigned to your Arduino.
```
avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
avr-objcopy blink.elf -O ihex blink.hex
```

If everything is working you should see something like.
```
$ avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
$ avr-objcopy blink.elf -O ihex blink.hex
```

## Programming the chip

At the time of writing WSL2 cannot *directly* interact with serial devices such as Arduinos or USB programmers. 

I've documented two workarounds:
- [Option 1: Using USB/IP](#option-1-using-usbip) - Use USB/IP to capture and expose the device in WSL.
- [Option 2: Using avrdude.exe (Windows-based)](#option-2-using-avrdudeexe-windows-based) - Use a Windows version of avrdude to program the chip since it can directly access the device.

These choices don't conflict with each other so you can always do both and see which you prefer.

### Option 1: Using USB/IP

Recent versions of Windows and WSL2 have support for accessing host USB devices using [usbipd-win](https://github.com/dorssel/usbipd-win). 

#### Install

To install, follow the instructions at (or see below for a shorter version): [https://devblogs.microsoft.com/commandline/connecting-usb-devices-to-wsl/](https://devblogs.microsoft.com/commandline/connecting-usb-devices-to-wsl/)

> If you run into issues, you may need to build a custom kernel for WSL which is covered in this guide. So far it hasn't been necessary for me.:
> - [https://github.com/dorssel/usbipd-win/wiki/WSL-support](https://github.com/dorssel/usbipd-win/wiki/WSL-support)


For convenience, I have summarized the steps below (see original guide for more information):

1. In WSL, run the following commands:
    ```bash
    sudo apt install linux-tools-5.4.0-77-generic hwdata
    sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/5.4.0-77-generic/usbip 20
    ```
1. In Windows Command Prompt (as Admin), run the following command:
    ```
    winget install --interactive --exact dorssel.usbipd-win
    ```

NOTE: After you've run it at least once as an Administrator in Windows, you can run it as a normal user in the future.  You can even run it from WSL. Notice how I've used the full filename including the extension (.exe) to tell windows we're running the windows executable):
```
usbipd.exe wsl attach --hardware-id=2341:0001
```

#### Test

To test the usbip setup, do the following:
1. Plug in a USB device to the Windows host
1. List the USB devices available:
    ```
    usbipd wsl list
    ```
1. You should see your device listed.  Note the bus ID (e.g 5-7)
1. Attach the device to WSL (update busid value with the value for your device):
    ```
    usbipd wsl attach --busid=5-7
    ```
1. In WSL, list the available usb devices:
    ```
    lsusb
    ```
1. You should see your device in list.

##### *Example 1: Arduino Uno*

Open an Administrative Command Prompt in Windows and attach the device to wsl using the commands below.  
```
C:\>usbipd wsl attach --hardware-id=2341:0001
usbipd: info: Device with hardware-id '2341:0001' found at busid '5-7'.
usbipd: info: Using default distribution 'Ubuntu'.
```
In this example, I'm attaching an Arduino Uno based on it's hardware ID. This value is fixed for a specific device. If you expect to have more than one of the same device attached at the same time, you'll need to use bus ID instead. For single device usage, hardware ID is more portable when you're using it for a specific device/programmer

Running lsusb and passing the same hardware ID should show the device is now available in WSL
```shell
$ lsusb -d 2341:0001
Bus 001 Device 002: ID 2341:0001 Arduino SA Uno (CDC ACM)
```

The Arduno Uno has attached itself as a CDC/ACM device, and so should be available via a /dev/ttyACM* device.

Running avrdude will require passing the appropriate ttyACM device path such as in the example below:
```bash
sudo avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P /dev/ttyACM0
```

##### *Example 2: USBasp*

Here is another example using a USBasp clone.

In Windows Command Prompt (as Admin), run the following command:
```
C:\>usbipd wsl attach --hardware-id=16c0:05dc
usbipd: info: Device with hardware-id '16c0:05dc' found at busid '5-7'.
usbipd: info: Using default distribution 'Ubuntu'.
```

In WSL, run the following command:
```shell
$ lsusb -d 16c0:05dc
Bus 001 Device 003: ID 16c0:05dc Van Ooijen Technische Informatica shared ID for use with libusb
```

Then in WSL run avrdude against that device. Notice we don't have to specify a port (-P) with usbasp:
```bash
sudo avrdude -c usbasp -p m328p -U flash:w:"blink.hex":a
```

Unless you're planning on using both options, you can skip ahead to [Next Steps](#next-steps).


> **Why do I need to use sudo with avrdude?**
>
> Normally in Ubuntu linux, when you plug in a device it run udev rules that setup the necessary device paths and the appropriate permissions.  Unfortunately in WSL2 that doesn't work fully because systemd isn't running. There are some workarounds that you can use to get this mostly working and you can read more about that in my other post, [Programming AVR in WSL Tips and Tricks]({% post_url 2022-05-20-programming-avr-in-wsl-tips-and-tricks %}).  Otherwise, you will need to run avrdude as root via sudo to gain access to the device.
{: .prompt-warning }


---


### Option 2: Using avrdude.exe (Windows-based)

This approach should work in most versions of Windows.

1. Install the Windows version of avrdude. See "Install AVR Dude" section of [Windows-based]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %}) tutorial.
1. Ensure avrdude.exe is on the PATH.

You can run avrdude from either Windows command prompt, or WSL, but if calling from WSL you need to include the full executable name including the extension -- avrdude.exe -- such as:

```
$ /mnt/c/AVR/avrdude/avrdude.exe -c arduino -p m328p -U flash:w:"blink.hex":a -P COM4

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

NOTE: Since you are calling the Windows version you should use the Windows method of identifying serial ports (e.g. COM4).


## Next Steps

That's all there is to it.  To continue setting up your development environment check out my other posts on building projects with a Makefile and configuring VS Code:
- [Building AVR projects with make]({% post_url 2022-05-22-building-avr-projects-with-make %})
- [Use VS Code with the AVR toolchain]({% post_url 2022-05-22-use-vscode-with-avr-toolchain %})

## Troubleshooting


How do I find out what COM port my Arduino is using?
: Open **Device Manager**. Expand **Ports (COM & LPT)**. You should see an entry named something like  "Arduino Uno (COM4)".  The COM port is in parentheses.

