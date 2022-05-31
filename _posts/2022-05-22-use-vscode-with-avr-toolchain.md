---
title: Use VS Code with the AVR toolchain
date: 2022-05-21 11:00:00 +0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, vscode]
---

## Overview

## Prerequisites

### Setup AVR toolchain

If you haven't already done so, start by setting up the AVR tool chain on your system so you can build AVR projects.

Follow the appropriate guide for your platform:
- [Windows-based AVR toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %})
- [WSL-based AVR toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})
- [Linux-based toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})

### Install Visual Studio Code
Download and install Visual Studio Code from [https://code.visualstudio.com/download](https://code.visualstudio.com/download)

### Install C/C++ extension pack
Install [C/C++ extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)

### Review Visual Studio Code C/C++ documentation
If you're not familiar with using VS Code with C/C++ on your chosen platform (Windows, WSL, Linux) I encourage you read through the VS Code documentation first before continuing.

Basic configuration
- [Using C++ on Linux in VS Code](https://code.visualstudio.com/docs/cpp/config-linux)
- [Using C++ and WSL in VS Code](https://code.visualstudio.com/docs/cpp/config-wsl)
- [Using GCC with MinGW](https://code.visualstudio.com/docs/cpp/config-mingw)

Cross-compiling (which is what we're doing with AVR)
- [IntelliSense for cross-compiling](https://code.visualstudio.com/docs/cpp/configure-intellisense-crosscompilation)


## Edit C configuration

Open the Command Palette (`Ctrl+Shift+P`)

Search for and run "C/C++: Edit Configurations (JSON)".
This will create a new c_cpp_properties.json file in .vscode/ directory in your project directory and open it for editing.



You will need to make the following general changes (see platform specific settings below):
- includePath: Add the path to the avr/include directory from the AVR toolchain.
- compilerPath: Update to avr-gcc (see platform specific settings below)
- intelliSenseMode: Update to {os}-gcc-{arch} ("windows-gcc-x64" or "linux-gcc-x64" most likely)
- compilerArgs: Add compiler arguments for mcu, F_CPU define, and optimization (-Os)


WSL / Linux settings:
- Include path: We installed this earlier during setup -- avr-libc in linux
    - e.g. /usr/lib/avr/include/
    - You can find the path with `dpkg -L avr-libc | grep avr/io.h`
- Change  "intelliSenseMode" to "linux-gcc-x64"
- Update the compilerPath to "/usr/bin/avr-gcc"

Windows settings:
- Include path: This is under the avr8-gnu-toolchain directory we created in the install
    - e.g. C:\AVR\avr8-gnu-toolchain\avr\include\
- Change  "intelliSenseMode" to "windows-gcc-x64"
- Update the compilerPath to avr-gcc.exe 
    - e.g. C:\AVR\avr8-gnu-toolchain\bin\avr-gcc.exe


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

## Variations

make via WSL from Windows
```
wsl make all
```

access to windows maked tools
```powershell
"Setting AVR build paths"
$toolpath = "C:\AVR"
$Env:PATH += ";$toolpath\avr8-gnu-toolchain\bin"
$Env:PATH += ";$toolpath\avrdude"
```

## Intellisense
Setting up intellisense in VS Code

Open folder.
Notice intellisense can't detect headers


