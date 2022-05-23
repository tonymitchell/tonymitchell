---
title: How to setup the AVR toolchain on Linux
date: 2022-05-21 10:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, avrdude, avr-gcc]
---

> ## Tutorial Versions
> 
> I have written 3 different versions of this tutorial that cover the following different setups:
> 1. [Windows-based]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %})
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
> 3. **Linux-based** (*this guide*)
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

