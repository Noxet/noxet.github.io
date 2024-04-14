---
title: "Zig Hello World on Stm32"
date: 2024-04-01T16:17:30+02:00
draft: true
---

How to use Zig in an embedded enrivonment?

# Setup
1. Install zig
    - Download a pre-built binary for Zig. At the time of writing, the microzig framework only works with zig-0.11,
    so if you experience issues with newer versions of Zig, make sure to have version 0.11 at hand.
1. Install stlink for Zig
    - Clone the forked stlink repo for Zig
    - Install prerequisites
    - Build it: `make release`
    - Add the `build/Release/bin` to PATH unless you want to install it system wide, then just run `make install`
