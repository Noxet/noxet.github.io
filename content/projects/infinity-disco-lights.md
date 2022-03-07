---
title: "Infinity Disco Lights"
date: 2022-03-07T00:31:56+01:00
draft: false
cover: "/projects/img/idl.jpg"
---

This is the "Infinity Disco Lights", or "how to abuse a digital counter".

Who doesn't like circuits with LEDs, especially if they flash in a cool, let's say infinity, pattern?
Well, I do.

Since music is one of my passions, I wanted to build something that connects the two. Enters the disco light.
This neat circuit listens to music via an electret microphone, and flashes the LEDs in an infinity pattern following the rhythm.

# The counter
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

# Using music as a clock signal
Now, the counter expects a clear square wave input at the clock. Unless you listen to some heavily synthesized music, this is not what we have.
Instead, we have sine waves, at frequencies spanning from 20Hz to 20kHz, not really an ideal clock signal.

However, just because a circuit expects a signal to look and behave in a certain way, doesn't mean that we need to conform to it to get it to work,
even though the result may be less than ideal.
Instead of a clean square wave, let's just use the music's sine waves to trigger the counter. This presents two problems:
- The input needs to have an amplitude large enough to trigger the counter into counting.
- The input signal needs to be in the range -0.5V to V<sub>DD</sub> + 0.5V, according to the documentation.

The first problem is solved by amplifying the signal from the microphone. 
The electret microphone is a condenser mic with a [FET](https://en.wikipedia.org/wiki/Field-effect_transistor) output buffer.
This means that we need a pull-up resistor to bias the transistor, as shown below. There is also a capacitor in series with the output,
to only get the AC signal, since we do not want to amplify the DC bias in the next stage.

```
 V_DD
  |
  _
 | | 10k
 |_|       100n
  |-------- || ---- Out
  |
 ,-,  |
(MIC) |
 '-'  |
  |
  |
 GND
```
The output from the mic goes in to a two-stage amplifier, since only a single stage was not enough to bring the signal up to a decent level.
The first stage amplifier is a common emitter (CE) amp. The bias is a simple resistor divider, setting the base bias to 0.82V.
The emitter resistor R<sub>E</sub> is 1k, setting the transistor quiescent current (I<sub>Q</sub>) to (0.82 - 0.6) / 1k = 0.22mA. With a collector resistor, R<sub>C</sub>, of
33k, the quiescent voltage (V<sub>Q</sub>) is 1.7V. The first stage including bias is shown below.

```
 +9V       +9V
  |         |
  _         _
 | | 100k  | | 33k
 |_|       |_| 
  |         |C
  |      B|/
  |-------|   BC547 NPN
  |       |\
  |         |E
  _         _
 | | 10k   | | 1k
 |_|       |_|
  |         |
  |         |
  |         |
 GND       GND
```

The second stage does not need to have its own bias network. Instead, we can use the DC voltage at the Q-point in stage one to bias the second amplifier.
Similar to stage one, we use a CE amp and set the Q-points to a reasonable value. As said before, amplification solves our first problem that the signal needs to have
a large enough amplitude to trigger the counter. 

To combat the second problem, that the input can not be negative, we need to set the output from our amplifier roughly
in the middle of the voltage swing. According to the documentation, there is a schmitt trigger at the clock input, allowing for fast rise and fall times, as well as some buffer
to keep the circuit stable. Powering the circuit with 10V sets the lower limit to 3V. This is the maximum voltage at which the circuit interprets the input as a zero (low).
The upper limit is 7V, where the circuit interprets the input as a one (high). Since a fully charged 9V battery reads at around 9.5V, it seems resonable to set the output
DC voltage between 4.5V - 5V. Now, as long as the amplitude goes beyond 3 and 7V, we are in the clear. Note that clean amplification is not needed here, we might as well overdo the amplification to make sure we hit well below 3V and well above 7V.

The amplification of a CE amp is determined by the collector and emitter resistors, Gain = R<sub>C</sub> / R<sub>E</sub>. This means that the amplification of our
circuit is fixed. This is not optimal, since the input signal from the mic is based on the volume of the music as well as the distance to the speaker.
It would be nice to let the user set the amplification based on the environment. Would a simple potentiometer do? Sadly no. Changing for example R<sub>E</sub> in
any stage would also change the Q-points (this is also true for R<sub>C</sub>). 

What to do then? Remember that we are only interested in amplifying AC signals, and not DC.
A capacitor is seen as infinite impedance to a DC signal, but a "normal" impedance for AC (although frequency dependent).
We can connect a capacitor in parallel to R<sub>E</sub> by replacing R<sub>E</sub> with a potentiometer, shown below.
```
  |
  |
  _
 | |_____
 |_|     |  
  |      |
  |     ---
  |     ---
  |      |
  |      |
 GND    GND
```
Notice here, that we do not change the resistance of R<sub>E</sub>, only how much we let AC signal bypass R<sub>E</sub>.
If the capacitor is all the way "down" to GND, it does not conduct, since we have the same voltage potential on both sides, so signals
has to flow through R<sub>E</sub>, and the gain is, as before, R<sub>C</sub> / R<sub>E</sub>.
If the capacitor is all the way "up", then AC signals bypass R<sub>E</sub> completely (a capacitor is basically a short circuit for AC).
This means that the AC gain is R<sub>C</sub> / R<sub>E</sub> = R<sub>C</sub> / 0 = infinity! Of course, in practice, the gain is far from infinity,
but let's save those details for another post.

All in all, we now have a way of controlling the gain, thus the sensitivity of the circuit, neat!
The complete amplifier circuit is shown below.

```
 +9V       +9V                       +9V
  |         |                         |                         
  |         |                         |                         
  _         _                         _                 CD4017B        
 | | 100k  | | 33k                   | | 22k         ____________           
 |_|       |_|                       |_|            |            |
  |         |---------------|         |-------------| Clock      |
  |         |C              |         |C            |____________|           
  |      B|/                |      B|/                          
  |-------|   BC547 NPN     |-------|   BC547 NPN               
  |       |\                        |\                          
  |         |E                        |E
  |         |                         |                        
  _         _ 1k                      _                         
 | | 10k   | |____                   | | 5.6k                     
 |_|       |_|    |                  |_|                        
  |         |    --- 10u              |                         
  |         |    ---                  |                         
  |         |     |                   |                         
 GND       GND   GND                 GND 
```
The output from the amp goes directly in to the clock of the counter.
The outputs from the counter is connected to 10mm super bright LEDs.

That's it!
The schematic and layout was done in Altium and can be found on my [GitHub](https://github.com/Noxet/infinity-disco-lights).
