---
title: Building AVR projects with make
date: 2022-05-21 11:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, make]
---

In this guide we'll go through the process of setting up a Makefile to build a simple AVR project.

This guide assumes you have already set up a working AVR toolchain.  If not please follow one of these guides first:
- [How to setup the AVR toolchain on Windows]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %})
- [How to setup the AVR toolchain on WSL2 (Windows 10+)]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})
- [How to setup the AVR toolchain on Linux]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})

## Project setup

Before we get into the Makefile, let's setup a basic project consisting of a sample AVR program, blink.c, in a new directory. This basic program will blink an LED on and off every 500ms.

1. Create a new directory named `blinker` to hold your project. It doesn't matter what it's named, but name it `blinker` so your your output matches the tutorial.
1. Create a new file named `blink.c`, and with the following contents:
    ```c
    #include <avr/io.h>
    #include <util/delay.h>

    int main()
    {
        // Set built-in LED pin as output
        DDRB |= (1 << DDB5);
        while (1) {
            PORTB |=  (1 << PB5);   // LED on
            _delay_ms(500);         // wait 500ms
            PORTB &= ~(1 << PB5);   // LED off
            _delay_ms(500);         // wait 500ms
        }
        return 0;
    }
    ```
    {: file="blink.c" }

## Building it manually

Now that we have the project set up, let's try building it manually first to see what's involved.

```console
$ avr-gcc blink.c -o blinker.elf -mmcu=atmega328p -DF_CPU=16000000UL -Os
```

If it was successful, you'll now have an compiled binary file along with your original .c file. You can remove that file before we continue: 
```console
$ rm blinker.elf
```

## Setting up a project Makefile

While you can certainly build the project like we did above, it error prone and more work especially when 

Rather than starting from scratch, I recommend using this [Makefile](https://github.com/hexagon5un/AVR-Programming/blob/master/setupProject/Makefile) as a starting point.  It was written by the author of the [Make: AVR Programming](https://www.oreilly.com/library/view/make-avr-programming/9781449356484/) book, and is a good place to get started. It only requires a few edits to adjust for your chip and programmer.

In your source folder, run the following commands to download the latest version of the Makefile, and make a few adjustments to it avoid errors since we're not using his full project template. 
```bash
# Download Makefile
wget https://raw.githubusercontent.com/hexagon5un/AVR-Programming/master/setupProject/Makefile

# Remove LIBDIR that we're not using
sed -i -e 's/^\(LIBDIR =.\+\)/\#\1 \# Not used/' -e 's/ -I$(LIBDIR)//' Makefile
```

## Configuring the Makefile for your chip and programmer

The next commands will configure the Makefile for building and flashing programs for the Arduino dev board.  If you're using a different chip or programmer you'll need to either adjust these commands or make the adjustments manually.  
```bash
# SETUP FOR ARDUINO
# Use ATmega328p and 16MHz with arduino programmer
sed -i -e 's/^MCU.*/MCU   = atmega328p/' -e 's/^F_CPU.*/F_CPU = 16000000UL/' Makefile
sed -i -e 's/^PROGRAMMER_TYPE.*/PROGRAMMER_TYPE = arduino/' -e 's/^PROGRAMMER_ARGS.*/PROGRAMMER_ARGS = -P \/dev\/ttyACM0/' Makefile
```


If you're adjusting the Makefile manually, these lines at the top of the file are the ones you need to edit. I'll explain them each briefly below:
```make
##########------------------------------------------------------##########
##########              Project-specific Details                ##########
##########    Check these every time you start a new project    ##########
##########------------------------------------------------------##########

MCU   = atmega328p
F_CPU = 16000000UL  
BAUD  = 9600UL
## Also try BAUD = 19200 or 38400 if you're feeling lucky.

## A directory for common include files and the simple USART library.
## If you move either the current folder or the Library folder, you'll 
##  need to change this path to match.
#LIBDIR = ../../AVR-Programming-Library

##########------------------------------------------------------##########
##########                 Programmer Defaults                  ##########
##########          Set up once, then forget about it           ##########
##########        (Can override.  See bottom of file.)          ##########
##########------------------------------------------------------##########

PROGRAMMER_TYPE = arduino
# extra arguments to avrdude: baud rate, chip type, -F flag, etc.
PROGRAMMER_ARGS = -P /dev/ttyACM0
```

MCU
: Defines what MicroController Unit (MCU) you are using. This variable is passed to both GCC and avrdude. For example, atmega328p is the MCU on an Arduino board. The complete list of options can be found in the [GCC documentation](https://gcc.gnu.org/onlinedocs/gcc/AVR-Options.html#AVR-Options). In addition to providing the compiler with instruction set and hardware details, this also sets up the appropriate `#define` values that control the AVR libary's header definitions for pins, ports, etc.

F_CPU
: Define the clock frequency of the MCU in Hz. Used by GCC. This value needs to match the fuse settings on your AVR chip. If not your program's timing will not be correct.  We're using 16 MHz (16,000,000 Hz) for the Arduino.

BAUD
: The baud rate for serial operations both by GCC for your program and avrdude for communication via serial programmers (if used in the PROGRAMMER_ARGS).

PROGRAMMER_TYPE
: Type of programmer you are using (e.g. arduino, usbasp, etc.). Used by avrdude to communicate with your chip. We are using arduino for the Arduino dev board.

PROGRAMMER_ARGS
: Additional arguments for avrdude.  Some programmers require additional arguments such as the serial port to use.  For example, when using arduino as the PROGRAMMER_TYPE we need to specify the serial port here (e.g. `-P /dev/ttyACM0`).


> **Configuration examples**
> 
> Example 1. Settings for building with the Arduino board:
> ```make
> MCU   = atmega328p
> F_CPU = 16000000UL  
> PROGRAMMER_TYPE = arduino
> PROGRAMMER_ARGS = -P /dev/ttyACM0
> ```
> 
> Example 2. Settings for building with the ATtiny85 and USBasp programmer:
> ```make
> MCU   = attiny85
> F_CPU = 8000000UL  
> PROGRAMMER_TYPE = usbasp
> PROGRAMMER_ARGS = 
> ```


## Executing the Makefile

Once we have the Makefile configured appropriately, we can run it using the `make` or `make all` command.  Both commands are equivalent for this Makefile as `all` is the first/default target defined in the Makefile.  Running either of these commands will build your project and produce an Intel hex (.hex) file that ready to be flashed to your chip.

```console
$ make all
```

The next step is to flash the chip using `make flash`. When using `make flash` it isn't necessary to run `make all` first since it will automatically execute the build step if anything has changed.

You will need to make sure that your Arduino (or other programmer) is connected to the computer. You may need to run this command with sudo if your user doesn't have the appropriate privileges to access the device.  See [Enable non-root user to program an Arduino](post_url 2022-05-21-enable-non-root-dev-board-access) to learn how to fix this.
```console
$ make flash
```

Another handy make target you can use is `make clean` or `make squeaky_clean` remove all the intermediate and final build output files.
```console
$ make squeaky_clean
```

