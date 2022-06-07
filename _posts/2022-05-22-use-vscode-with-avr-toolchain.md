---
title: Use VS Code with the AVR toolchain
date: 2022-05-21 11:00:00 -0700
categories: [Tutorial, AVR]
tags: [tutorial, avr, vscode]
---

## Overview

This guide will take you through the steps to configure Visual Studio Code for AVR programming in C/C++, including working Intellisense and a build task to execute your Makefile with different targets.

## Prerequisites

### Setup AVR toolchain

If you haven't already done so, start by setting up the AVR tool chain on your system so you can build AVR projects.

Follow the appropriate guide for your platform:
- [Windows-based AVR toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-windows %})
- [WSL-based AVR toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-wsl %})
- [Linux-based toolchain]({% post_url 2022-05-21-setup-avr-toolchain-on-linux %})

While you're at it you might as well head over and set up a Makefile to build your project now too as we'll need that later when we set up the build task:
- [Building AVR projects with make]({% post_url 2022-05-22-building-avr-projects-with-make %})

### Install Visual Studio Code
Download and install [Visual Studio Code](https://code.visualstudio.com/download)

### Install C/C++ extension pack
Install the [C/C++ extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)

### Review Visual Studio Code C/C++ documentation
If you're not familiar with using VS Code with C/C++ on your chosen platform (Windows, WSL, Linux) I encourage you read through the VS Code documentation first before continuing.

Basic configuration
- [Using C++ on Linux in VS Code](https://code.visualstudio.com/docs/cpp/config-linux)
- [Using C++ and WSL in VS Code](https://code.visualstudio.com/docs/cpp/config-wsl)
- [Using GCC with MinGW](https://code.visualstudio.com/docs/cpp/config-mingw) (I use gnuwin32 instead of MingGW)

Cross-compiling (which is what we're doing with AVR)
- [IntelliSense for cross-compiling](https://code.visualstudio.com/docs/cpp/configure-intellisense-crosscompilation)

## Open the sample project

Start by opening the folder for your AVR project in VS Code.  I'm going to use the blinker sample created in previous tutorials which consists of a folder named blinker, a single .c file named blink.c, and a Makefile.  If you don't already have a known working project to use, I suggest 

## Edit C/C++ configuration

Open the Command Palette (`Ctrl+Shift+P`)

Search for and run `C/C++: Edit Configurations (JSON)`.
This will create a new `c_cpp_properties.json` file in the `.vscode/` directory of your project and open it for editing.

You will need to make the following changes (see platform specific settings below):
- **compilerPath**: Update to avr-gcc (see platform specific settings below)

    |-|-|
    |`"-mmcu=atmega328p"`|Will ensure MCU defines are set correctly|
    |`"-DF_CPU=16000000UL"`|Will ensure F_CPU is set correctly|
    |`"-Os"`|Will avoid optimization warnings re: _delay|

- **intelliSenseMode**: Update to {os}-gcc-{arch} ("windows-gcc-x64" or "linux-gcc-x64" most likely)
- **compilerArgs**: Add any compiler arguments you use such as the mcu, F_CPU define, and optimization (-Os).  These are necessary for the compiler and VS Code to work together to produce the right set of macros #defines for your MCU.

> **includePath**: No need to specify the AVR include paths as they will be included automatically.
>
> From the [VS Code documentation](https://code.visualstudio.com/docs/cpp/configure-intellisense-crosscompilation#_include-path):
> *"You only need to modify the Include path if your program includes header files that aren't in your workspace, or that are not in the standard library path. The C/C++ extension populates the include path by querying the compiler specified by Compiler path. If the extension can't find the path for the target system libraries, you can enter the include path manually"*
{: .prompt-tip }

On WSL / Linux:
- Change  "intelliSenseMode" to "linux-gcc-x64"
- Update the compilerPath to "/usr/bin/avr-gcc"
- Include path: **(Only if necessary)**
    - e.g. /usr/lib/avr/include/
    - You can find the path by running the command:
        ```
        dpkg -L avr-libc | grep avr/io.h
        ```

On Windows:
- Change  "intelliSenseMode" to "windows-gcc-x64"
- Update the compilerPath to avr-gcc.exe 
    - e.g. C:\AVR\avr8-gnu-toolchain\bin\avr-gcc.exe
- Include path:  **(Only if necessary)**
    - This is under the avr8-gnu-toolchain directory we created in the install
    - e.g. C:\AVR\avr8-gnu-toolchain\avr\include\


When you're done it should like one of these examples below (depending on your platform).

On Linux:
```jsonc
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
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
{: file="c_cpp_properties.json" }

On Windows:
```jsonc
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"                
            ],
            "compilerPath": "C:/AVR/avr8-gnu-toolchain/bin/avr-gcc.exe",
            "cStandard": "gnu11",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "windows-gcc-x64",
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
{: file="c_cpp_properties.json" }

Now that the above configration has been completed, your Intellisense should be working and you could use the built-in gcc build tasks via **Run Build Task...** (`Ctrl+Shift+B`) to run a simple build, but we're going to use a `Makefile`. If you don't have a Makefile for your project, you can create one in [Building AVR projects with make]({% post_url 2022-05-22-building-avr-projects-with-make %}).  

In the next step we'll configure VS Code to build the project using your Makefile.

## Build tasks

Let's start by creating a `tasks.json` in our .vscode directory to define our new build task.  While you can do this manually, we're going to have VS Code generate the basic template for us.

1. From the Terminal menu, select **Configure Tasks...**.  Or search for **Configure Task** from the Command Palette (`Ctrl+Shift+P`)
1. Choose **Create tasks.json file from template**
1. Choose **Others**

Setting up a build task using make

```jsonc
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make",
            "detail": "Run make",
            "type": "shell",
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
{: file="tasks.json" }

## Variations

### Windows but using WSL for make

make via WSL from Windows
```
wsl make all
```

### Using Windows-based tools that aren't 

access to windows maked tools
```powershell
"Setting AVR build paths"
$toolpath = "C:\AVR"
$Env:PATH += ";$toolpath\avr8-gnu-toolchain\bin"
$Env:PATH += ";$toolpath\avrdude"
```





## Troubleshooting

VS Code can't find my AVR headers.
: 1. First check that your AVR headers are installed.  On Ubuntu Linux/WSL this is a separate package: `avr-libc`.  
  1. Next check that you've specified the compilerPath and compilerArgs appropriately as VS Code finds the include paths by interrogating the compiler.

VS Code STILL can't find my AVR headers.
: Add the path to the AVR include paths manually. 
    
    On Linux:
    - e.g. /usr/lib/avr/include/
    - You can find the path by running the command:
        ```console
        dpkg -L avr-libc | grep avr/io.h
        ```
    
    On Windows:
    - This is under the avr8-gnu-toolchain directory we created in the install
    - e.g. C:\AVR\avr8-gnu-toolchain\avr\include\

It's still not working
: Check out the [VS Code FAQ](https://code.visualstudio.com/docs/cpp/faq-cpp) for more help.