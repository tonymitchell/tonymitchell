---
title: How to manually install AVR tools on Linux (or WSL2)
date: 2022-05-21 10:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, avrdude, avr-gcc]
---

If you're just looking to install the AVR tools on Linux or WSL please try one of these approaches first:
- [How to setup the AVR toolchain on Linux]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})
- [How to setup the AVR toolchain on WSL2 (Windows 10+)]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})


If for some reason, the Ubuntu packaged versions don't work for you, it is possible to manually install the latest versions of the tools.  This post will guide you through that process. I don't recommend this approach, but if you're experiencing an issue with an older version and need the latest version you can follow some or all of these steps to install the latest version.  NOTE: If you're on a recent version of Ubuntu, avr-gcc tends to be up-to-date. It's only avrdude that doesn't include the latest version.

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
