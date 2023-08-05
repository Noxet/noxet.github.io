---
title: "UPDIY"
date: 2022-03-07T00:31:56+01:00
draft: true
cover: "/projects/img/idl.jpg"
---

UPDIY - How to build your own UPDI programmer for AVR tiny 0/1 series.

# Set up the toolchain
If you are using the latest Microchip (Atmel) Studio or MPLAB, everything should already be set up to work without further
adjustments.

This post is about how to set up the AVR gcc toolchain along with avrdude.

## Compiler and libc

However, using avr-gcc (and its avr-libc) proved to be a bit more painful (running on macOS 13.1).
The avr-gcc from `brew` (verion 9.3.0 at the time of writing) does not contain `libc.a` and `libm.a` which is needed
by the linker. Therefore as a starting point:
- Download the latest toolchain from Microchip (avr-gcc 7.3.0) 
- Extract it to a good location
- Set the PATH variable to point to <install folder>/bin, which contains all tools.

The included libc (from Microchip) did not contain information about the new series of ATtinys except for the 1-series. This is confirmed when trying
to compile a simple "hello world" program:
```sh
> avr-gcc -mmcu=attiny402 main.c
avr/io.h:581:6: warning: #warning "device type not defined"
```
What this means is that the included `io.h` file does not have a definition for the attiny402, in this case.

To resolve this issue, we can download the device support pack from [Microchip](https://packs.download.microchip.com/)
In this case, I downloaded the support pack for the ATtiny series.
Even though the file extension is `.atpack` it is actually a zip file, so go ahead and extract it.

Now we can copy the missing files to the installation folder of `avr-gcc`:
- `cp include/avr/iotn* <install folder>/avr/include/avr/` - The device-specific header files included by `avr/io.h`
- `cp gcc/dev/attiny*/avrxmega3/*.{a,o} <install folder>/avr/lib/` - The C-runtime library
- `cp gcc/dev/attiny*/avrxmega3/short-calls/*.{a,o} <install folder>/avr/lib/` - The C-runtime library for MCUs with under 8k memory (using short-call instructions)
- `cp attiny*/device-specs/* <install folder>/lib/gcc/avr/7.3.0/device-specs` - The device specifications used by the compiler
*Note* that I ran this commands in `zsh`, so the wildcards might not work for you. Worst case, you have to copy them
manucally, either all or just the ones you need.

Then, we need to update `avr/io.h` to include the correct headers:
- Either do it manually, following the structure of the header file, adding the new MCUs you want
- or, copy my header file [LINK] and replace the current one (`<install folder>/avr/include/avr/io.h`)

## avrdude

In order to actually program our devices, we use avrdude. In order to use avrdude with jtag2updi, we need to do
some modifications. We need to update the avrdude configuration file to include the "new" programmer jtag2updi.
From the [jtag2updi repo](https://github.com/ElTangas/jtag2updi/blob/master/README.md#using-with-avrdude), 
we can get the updated conf file. Just copy the file from the root of the repo, and either replace the original one
or create a local version in `~/.avrduderc`.

Now, we should be able to read the MCU signature by running
```sh
> avrdude -c jtag2updi -p t402 -P /dev/<USB>

avrdude: AVR device initialized and ready to accept instructions
avrdude: device signature = 0x1e9227 (probably t402)
```
