---
title: "Infinity Disco Lights"
date: 2022-03-04T00:31:56+01:00
draft: true
cover: "/projects/img/idl2.jpg"
---

This is the "Infinity Disco Lights", or "how to abuse a digital counter".

Who doesn't like circuits with LEDs, especially if they flash in a cool, let's say infinity, pattern?
Well, I do.

Since music is one of my passions, I wanted to build something that connects the two. Enters the disco light.
This neat circuit listens to music via an electret microphone, and flashes the LEDs in an infinity pattern following the rhythm.

# The Counter
The circuit is built around a digital decade counter, namely the [CD4017B](https://www.ti.com/lit/ds/symlink/cd4017b.pdf) IC.
This counter is commonly used in applications to perform frequency division, divide-by-N counter, decimal decode displays and so on.
The IC has the following inputs:
- Clock: This is the clock input where each rising edge advances the counter by one.
- Clock inhibit: This input controls whether to advance the counter or to freeze it.
- Reset: Resets the chip to the starting state where pin 0 is high.

A simple diagram showing the counter in action, assuming reset and clock inhibit are low:
```
          ___     ___     ___     ___     ___     ___ 
Clock ___|   |___|   |___|   |___|   |___|   |___|   |___
          _______
"0"   ___|       |_______________________________________
                  _______ 
"1"   ___________|       |_______________________________
                          _______ 
"2"   ___________________|       |_______________________
                                  _______ 
"3"   ___________________________|       |_______________
...
```

The maximum input clock frequency, running at 10V, is 5MHz. This is well above the required 20kHz that we are dealing with.

# The Trick
Now, the counter expects a clear square wave input at the clock. Unless you listen to some heavily synthesized music, this is not what we have.
Instead, we have sine waves, at frequencies spanning from 20Hz to 20kHz, not really an ideal clock signal.

