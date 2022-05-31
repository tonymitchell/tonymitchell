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


