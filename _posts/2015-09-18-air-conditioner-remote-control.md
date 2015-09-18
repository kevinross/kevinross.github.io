---
layout: post
title: "Air Conditioner Remote Control"
description: "One of my first steps into home automation"
category: 
tags: ["home automation", "raspberrypi"]
---
{% include JB/setup %}

I picked up an [air conditioner](https://www.lowes.ca/air-conditioners/honeywell-mm14ccs-portable-air-conditioner-14000-btu_g1616504.html) a few months back, been wanting one for a while but only was now able to afford one. One thing bugged me though: while it had a remote control, there was no way to integrate it into any sort of home automation system: the remote uses a proprietary protocol that sends the entire state on each button press. So began my journey to integrate it.

## First Failure: Reverse Engineering the Protocol

After scouring the internet, several other people had the same idea and were able to successfully reverse engineer the IR protocol and reproduce signals to set the A/C's state. I opted to go this route as I'd be able to send arbitrary commands to the unit and not have to track state myself unnecessarily.

### Recording the Signals
 
I looked around my apartment for an IR receiver I could salvage and couldn't find any so I went and wired the IR led's pins to GPIO+GND on a RPi2. *Should* effectively be the same however no tool for IR signal capture was able to use it nor was the carrier filtered out by the non-existent IR receiver/software. I *was* able to get mode2 (in lirc) to record the raw signals from GPIO. Once that was done, I derived the various pieces involved (carrier frequency, header, signal lengths, etc).
 
### Generating the Signals

I attempted to reproduce the signal with an IR led I ordered from Amazon. No joy. I suspect either the wavelength of the IR led is incorrect or the signals I generated are incorrect (literally realized as I write this that I can test that pretty quickly... hindsight, fun, right? More on this later).

This would've been the ideal route as I wouldn't have to solder anything to the remote, potentially destroying it in the process, nor track the likely state of the remote as buttons are "pressed" by relays on a breadboard with a RPi.

## Second Failure (well, almost one, but it's 60% successful): Soldering Is Hard

### Soldering

Next shot was to solder wires to each side of the button traces on the remote so I could stick a relay in controlled by a RPi. After a not-so-quick examination, I found I was able to halve the number of wires I'd need as the board multiplexes 2 buttons for each line from the MCPU. Yay, less soldering! A [circuit diagram](https://github.com/kevinross/homeautomation/blob/master/ac/diagram.py).

I haven't soldered a thing in my life before now so it was interesting going from nothing to soldering wires to sub-millimetre width traces on a circuit board. They broke off many, many times before I was finally able to get them to stick (even now they're fragile).

### Testing (the fail)

Wired everything up (the [fritzing sketch](https://github.com/kevinross/homeautomation/blob/master/ac/ac_remote.fzz) I made for reference) and wrote a quick [python program](https://github.com/kevinross/homeautomation/blob/master/ac/test.py) to trigger the relays conveniently (maps keypresses for 1-5 to buttons). 2 buttons weren't working, temperature up and speed. Back to the drawing board.

## Success!

I miswired some of the buttons to the relays so 2 were basically no-oping. Once I fixed that, everything worked great! Yeah, I need to track state of the remote but I plan to add one more relay to control power to the remote so I can reset it to a known state (it defaults to 76F, AC, full speed on power loss).

# Home Automation

I haven't put together any sort of proper system to work this in to my apartment but I already have a few pieces: an extra RPi and [touchscreen](http://www.adafruit.com/product/1983) for a controller and a [tasker](http://tasker.dinglisch.net/) profile for location.
 
## RPi and Touchscreen

This RPi will be the hub to control "things" in my apartment. Right now, that'll be the air conditioner and a remote controlled power outlet (used to power cycle my router, more on that in another post).

## Tasker Profile

I've a tasker profile setup using the [AutoLocation](http://joaoapps.com/autolocation/) plugin so that when I get within or leave a certain radius of my apartment, I can turn the air conditioner on or off. 

# Next Steps

I mentioned I realized IR signal generation may not be a lost cause; if the reason it wasn't working is due to wavelength, I can still attempt to generate IR signals and send them **via the remote's own IR led**! This would be about the closest to ideal I can get without actually desoldering the IR led and potentially destroying it (and the remote) in the process (something I'm not willing to do).

A nice bonus of having wired the buttons up as they are: I can automate gathering more samples from the IR led for analysis (by programmatically pressing buttons)!


*NB: running short on time while writing this up so the code+diagrams linked aren't published yet but will be shortly*