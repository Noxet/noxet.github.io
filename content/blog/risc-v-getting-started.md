---
title: "Getting Started with Risc-V"
date: 2024-03-10T23:48:50+01:00
oneHeadingSize: false
draft: true
---

Today, we take a look at getting a cheap Risc-V microcontroller up and running free and open tools.
For this example, we use a CH32V MCU (Ch32V003F4P6) together with a programmer, the WCH-Link??.

# Toolchain Setup
Let's dive right into it and get the toolchain setup so we can compile and flash our MCU.

## GCC

## OpenOCD
Download MounRiver, in the OpenOCD folder we find a pre-built CH32V compatible OpenOCD version
In the "beforeinstall" folder, there are some udev rules along with .so files needed for OpenOCD.
(They can be installed via apt in case we don't trust the pre-built ones).
There is also a [GitHub repo](https://github.com/kprasadvnsi/riscv-openocd-wch) containing the (supposedly) source code for the CH32V version of OpenOCD

### If running inside WSL

#### USB Access

To access the physical USB device under WSL, we need to install the [usbipd-win](https://github.com/dorssel/usbipd-win) tool as denoted by [Microsoft](https://learn.microsoft.com/en-us/windows/wsl/connect-usb).

To attach the COM port inside WSL, we need to do:
- `usbipd list` to list all available USB devices (make sure to install the driver for the WCH-Link).
- `usbipd bind --busid x-y`, where "x-y" is the bus ID shown in the listing for the WCH-Link
- `usbipd attach --wsl --busid x-y` to attach it to the WSL session (we can no longer access it from Windows)

The device should now be visible in WSL, check with `lsusb`.

To detach it and give back access to Windows:
- `usbipd -- detach --busid x-y`

#### Udev rules
Make sure to start udev: `sudo service udev start`, or set it up during boot by adding the following to `/etc/wsl.conf`:
```
[boot]
command="service udev start"
```

# Hello, World!
Now it's time to actually write some code and upload it to the MCU! :)

## Flash
To flash the firmware, we need a configuration file for OpenOCD specifically for our MCU. Luckily, WCH provides one, and it is found in the [MRS Toolchain](http://www.mounriver.com/download) archive for Linux. It is located in the OpenOCD/bin folder.

To flash the firmware, simply run
```
openocd -f wch-riscv.cfg -c init -c halt -c "program <myproject>.hex" -c exit
```
replace "<myproject>" with the actual name of the hex file