---
title: Use VS Code with the AVR toolchain
date: 2022-05-21 11:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, vscode]
---

## Overview

## Setup AVR toolchain
See [How to setup the AVR toolchain on Windows]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %})

## Install VS Code
Download and install VS Code from [https://code.visualstudio.com/download](https://code.visualstudio.com/download)

## Install C/C++ extension pack
Install [C/C++ extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)

## Project setup

Before we continue let's setup a basic project consisting of a sample AVR program, blink.c, and a Makefile.

Create a new directory for our project. It doesn't matter what it's named, but let's called it `blink`

In the blink directory, create a new file named blink.c with the following contents:
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

Open the `blink` directory in VS code.
```console
~/blink$ code .
```

## Setting up a project Makefile
You'll need a Makefile to build your program and program the chip.

Rather than starting from scratch, I recommend using this [Makefile](https://github.com/hexagon5un/AVR-Programming/blob/master/setupProject/Makefile) as a starting point.  It was written by the author of the AVR Programming book, and is a good place to get started. It only requires a few edits to adjust for your chip and programmer.

In your source folder, run the following commands to download the latest version of the Makefile, and make a few adjustments to it avoid errors since we're not using his full project template. 
```bash
# Download Makefile
wget https://raw.githubusercontent.com/hexagon5un/AVR-Programming/master/setupProject/Makefile

# Remove LIBDIR that we're not using
sed -i -e 's/^\(LIBDIR =.\+\)/\#\1 \# Not used/' -e 's/ -I$(LIBDIR)//' Makefile

# Use ATmega328p and 16MHz with arduino programmer
sed -i -e 's/^MCU.*/MCU   = atmega328p/' -e 's/^F_CPU.*/F_CPU = 16000000UL/'
sed -i -e 's/^PROGRAMMER_TYPE.*/PROGRAMMER_TYPE = arduino/' -e 's/^PROGRAMMER_ARGS.*/PROGRAMMER_ARGS = -P \/dev\/ttyACM0/' Makefile
```




Try building the project manually with make:
```console
$ make
```

Ensure your Arduino is plugged in.  We need to run make using sudo right now because we haven't granted the appropriate permissions to your user.
```console
$ sudo make flash
```

## Edit C configuration

Ctrl + Shift + P
Search for and run "C/C++: Edit Configurations (JSON)".
This will create a new c_cpp_properties.json file in .vscode/ directory in your project directory and open it for editing.

Make the following changes:
- Add the path to the avr/include directory from the AVR toolchain. (We installed this earlier during setup -- avr-libc in linux)
    - You can find the path with `dpkg -L avr-libc | grep avr/io.h`
- Update the compilerPath to avr-gcc
- Change  "intelliSenseMode" to "windows-gcc-x64"
- Add compilerArgs for mcu, F_CPU define, and optimization (-Os)

Linux:
```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/lib/avr/include/"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/avr-gcc",
            "cStandard": "gnu11",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "linux-gcc-x64",
            "compilerArgs": [
                "-mmcu=atmega328p",     // Will ensure MCU defines are set correctly
                "-DF_CPU=16000000UL",   // Will ensure F_CPU is set correctly
                "-Os"                   // Will avoid optimization warnings re: _delay
            ]
        }
    ],
    "version": 4
}
```

Windows:
```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
                "C:/AVR/avr8-gnu-toolchain/avr/include"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"                
            ],
            "windowsSdkVersion": "10.0.19041.0",
            "compilerPath": "C:/AVR/avr8-gnu-toolchain/bin/avr-gcc.exe",
            "cStandard": "c11",
            "cppStandard": "c++11",
            "intelliSenseMode": "windows-gcc-x64",
            "compilerArgs": [
                "-mmcu=attiny85",       // Will ensure MCU defines are set correctly
                "-DF_CPU=8000000UL",    // Will ensure F_CPU is set correctly
                "-Os"                   // Will avoid optimization warnings re: _delay
            ]
        }
    ],
    "version": 4
}
```


## Build tasks
Setting up a build task using make

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "shell",
            "label": "make",
            "detail": "Run make",
            "command": "${input:makeTarget} ",
            "options": {
                "env": {
                }
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build"
            }
        }
    ],
    "inputs": [
        { 
            "type": "pickString", 
            "id": "makeTarget", "description": "Select a build target", 
            "options": [
                "make all", "sudo make flash", "make disassemble", "make squeaky_clean", "make size", "make debug",
            ],
            "default": "make all"
        }
    ]
}
```


## Intellisense
Setting up intellisense in VS Code

Open folder.
Notice intellisense can't detect headers


