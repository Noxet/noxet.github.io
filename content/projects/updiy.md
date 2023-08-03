---
title: "UPDIY"
date: 2022-03-07T00:31:56+01:00
draft: false
cover: "/projects/img/idl.jpg"
---

UPDIY - How to build your own UPDI programmer for AVR tiny 0/1 series.

# Set up the toolchain
If you are using the latest Atmel Studio, everything should already be set up to work without further
adjustments.

However, using avr-gcc (and its avr-libc) proved to be a bit more painful (running on macOS 13.1).
The included libc did not contain information about the new series of MCUs. This is confirmed when trying
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
- `sudo cp include/avr/iotn* <install folder>/avr/include/avr/`

Then, we need to update `avr/io.h` to include the correct headers:
- Either do it manually, following the structure of the header file, adding the new MCUs you want
- or, copy my header file [LINK] and replace the current one (`<install folder>/avr/include/avr/io.h`)


