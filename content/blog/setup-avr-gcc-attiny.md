---
title: "Setup avr-gcc for the new ATtiny 0/1/2 series"
keywords: ["attiny", "microchip", "avr", "c", "c++", "embedded", "mcu", "microcontrollers"]
description: "A tutorial on how to set up avr-gcc toolchain for development on the new ATtiny series of microcontrollers"
date: 2023-08-05T00:32:54+02:00
draft: false
readingTime: true
---

# Background
The new ATtiny series is a nice addition to the AVR family.
However, it is not as trivial to just run avr-gcc as usual and believe that stuff
will just... work... that would be all too easy :)

Here, I describe my own journey trying to get all this stuff working, so that I can use the toolchain and 
development setup I have always used.

# Set up the toolchain
If you are using the latest Microchip (Atmel) Studio or MPLAB, everything should already be set up to work without further
adjustments.

This post is about how to set up the AVR GCC toolchain along with avrdude.

## Compiler and libc

Just using avr-gcc (and its avr-libc) proved to be a bit more painful (running on macOS 13.1).
The avr-gcc from `brew` (verion 9.3.0 at the time of writing) does not contain `libc.a` and `libm.a` which is needed
by the linker. Therefore as a starting point:
- Download the [latest toolchain](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers) from Microchip (avr-gcc 7.3.0) 
- Extract it to a good location
- Set the PATH variable to point to \<install folder\>/bin, which contains all tools.

The included libc (from Microchip) did not contain information about the new series of ATtinys except for the 1-series. This is confirmed when trying
to compile a simple "hello world" program:
```sh
> avr-gcc -mmcu=attiny402 main.c
avr/io.h:581:6: warning: #warning "device type not defined"
```
What this means is that the included `io.h` file does not have a definition for the MCU, attiny402 in this case.

To resolve this issue, we can download the [device support pack](https://packs.download.microchip.com/) from Microchip.
In this case, I downloaded the support pack for the ATtiny series.
Even though the file extension is `.atpack` it is actually a zip file, so go ahead and extract it.

Now we can copy the missing files to the installation folder of `avr-gcc`:
- `cp include/avr/iotn* <install folder>/avr/include/avr/` - The device-specific header files included by `avr/io.h`
- `cp gcc/dev/attiny*/avrxmega3/*.{a,o} <install folder>/avr/lib/avrxmega3/` - The C-runtime library
- `cp gcc/dev/attiny*/avrxmega3/short-calls/*.{a,o} <install folder>/avr/lib/avrxmega3/short-calls` - The C-runtime library for MCUs with under 8k memory (using short-call instructions)
- `cp attiny*/device-specs/* <install folder>/lib/gcc/avr/7.3.0/device-specs` - The device specifications used by the compiler

*Note* that I ran this commands in `zsh`, so the wildcards might not work for you. Worst case, you have to copy them
manually, either all or just the ones you need.

Then, we need to update `avr/io.h` to include the correct headers:
- Either do it manually, following the structure of the header file, adding the new MCUs you want
- or, download my [header file](/io.h) and replace the current one (`<install folder>/avr/include/avr/io.h`)
