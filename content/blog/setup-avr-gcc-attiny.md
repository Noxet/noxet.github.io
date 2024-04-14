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

Now we can copy the missing files to the installation folder of `avr-gcc`.

- The device-specific header files included by `avr/io.h`:
```sh
cp include/avr/iotn* <install folder>/avr/include/avr/
```
- The C-runtime library:
```sh
cp gcc/dev/attiny*/avrxmega3/*.{a,o} <install folder>/avr/lib/avrxmega3/
```
- The C-runtime library for MCUs with under 8k memory (using short-call instructions):
```sh
cp gcc/dev/attiny*/avrxmega3/short-calls/*.{a,o} <install folder>/avr/lib/avrxmega3/short-calls
```
- The device specifications used by the compiler:
```sh
cp gcc/dev/attiny*/device-specs/* <install folder>/lib/gcc/avr/7.3.0/device-specs
```

*Note* that I ran this commands in `zsh`, so the wildcards might not work for you. Worst case, you have to copy them
manually, either all or just the ones you need.

Finally, we need to update `avr/io.h` to include the correct headers, by adding defines and includes for out target
MCUs:
- Either do it manually, following the structure of the header file, adding the new MCUs you want
- or, download my [header file](/io.h) and replace the current one (`<install folder>/avr/include/avr/io.h`)

# Building hello world

To verify that our setup actually works, let's create a simple hello world program and compile it.
```c {linenos=table}
#define F_CPU 20000000ULL

#include <avr/io.h>
#include <util/delay.h>

int main(void)
{
    // configure clock, and disable prescaler
    CCP = CCP_IOREG_gc;
    CLKCTRL.MCLKCTRLB &= ~(1 << 0);

    // setup LED pin
    PORTA.DIRSET = PIN2_bm;

    while (1)
    {
        PORTA.OUTSET = PIN2_bm;
        _delay_ms(500);
        PORTA.OUTCLR = PIN2_bm;
        _delay_ms(500);
    }
}
```

Compile with
```sh
avr-gcc --mmcu=attiny402 -O2 main.c -o main.elf
```

We can check the section sizes:
```sh
> avr-size main.elf
   text	   data	    bss	    dec	    hex	filename
    140	      0	      0	    140	     8c	main.elf
```

We can also generate then hex file used for flashing:
```sh
> avr-objcopy -j .data -j .text -O ihex main.elf main.hex
```

That's it!
You should now have a fully working toolchain for the latest MCUs.

Check out the coming posts about setting up avrdude, and how to build your own UPDI programmer.
