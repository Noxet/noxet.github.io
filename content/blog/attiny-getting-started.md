+++
title = "Getting Started with ATtiny 0/1/2 series"
date = "2023-08-05T00:34:10+02:00"
author = "noxet"
authorTwitter = "" #do not include @
cover = ""
tags = ["", ""]
keywords = ["attiny", "microchip", "avr", "c", "c++", "embedded", "mcu", "microcontrollers"]
description = "A tutorial to get started with development on the new ATtiny series of microcontrollers"
showFullContent = false
readingTime = false
draft = true
+++

# Intro
Continuing our journey with the "new" tiny series, we should have our toolchain properly set up by now.
If you have not set it up, see the previous blog post {{< ref "setup-avr-gcc-attiny.md" >}}

# Flashing
The easiest way to flash the devices is by skipping avrdude (:/) and use a tool by Microchip called pymcuprog. (link GH)
The only hardware we need is a FTDI chip (or other TTL serial) converting from USB to Serial. Reading the docs for pymcuprog, we see that
we need to use the switch `-t uart` for flashing the MCU over a serial port, neat! (https://github.com/microchip-pic-avr-tools/pymcuprog?tab=readme-ov-file#serial-port-updi-pyupdi)

To flash, we can write
```sh
pymcuprog write -f main.hex -d attiny402 -t uart -u /dev/<your tty device> --erase --verify
```
