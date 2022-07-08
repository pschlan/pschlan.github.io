---
layout: post
title:  "Modifying a Samsung Blu-ray player to run on 12V in a camper"
date:   2022-07-07 21:15:00 +0200
categories: howtos
---

My parents bought a camper / mobile home which has a TV running on its internal 12V supply. For rainy days, they wanted to add a Blu-ray player to watch movies, but unfortunately the 230V supply is only available when the camper is connected to a power supply and not when running in self-sustaining mode using its solar panels.

They have a _Samsung BD-F5100/EN_ and I got curious if it's possible to modify it to run off a 12V supply so they can watch movies with no mains supply around.

> :warning: This is just a report of what I changed in my own device. **Only do this if you're absolutely sure you know what you're doing.** Disconnect the power plug first and wait at least 30 minutes to allow all capacitors to be discharged. Ensure the devices stays unplugged while you're working on it. Improper wiring, bad connections and missing or not correctly sized fuses might cause damage and fire. I don't accept any liability for damages or injuries. Needless to say, your warranty will void when opening/modifying the player. Everything's on your own risk!

# What's inside?

After opening the case of the device, one can see two separate circuit boards, one SMD-style multi-layer one containing many ICs including the main SoC and one with rather large discrete components which serves as the power supply.

The power supply PCB doesn't only contain the power supply itself, but also the front plate components including the infrared sensor for the remote control, a USB port and the interface for the buttons located on top of the device which can be used when there's no remote around.

Fortunately, the power supply PCB has some nice labels printed on it. The pin header connecting it to the main PCB is fully documented and we can see that the power supply provides a +12V and a 3.3V lane to the main PCB. I suspect the +12V lane is for powering the Blu-ray drive while the +3.3V lane is providing power for the main SoC.

# Modification for 12V

The +12V lane is an easy job: We can just use the 12V camper supply to power it - provided we're sure that the supply voltage is stable enough and there are no over-voltage conditions. In the given camper, it's powered from a 12V battery, so I'm pretty confident there will be no over-voltage.

The +3.3V lane is a bit harder and requires a step-down converter. After browsing Amazon for a while, I've decided to give [this one](https://www.amazon.de/gp/product/B09DCHWZ2X/){:target="_blank"} a try, mostly because it comes in a nice case, is very compact and provides up to 3A of current. (The back side of the Blu-ray player claims it's consuming 10W at max, so I suppose 3A are enough for the 3.3V lane: `3A * 3.3V = 9.9 W`.)

Summarizing, the plan is as follows:

![Wiring diagram](/assets/2022/07/bd_wiring.png)

# Wiring it up

Following the +12V, +3.3V and GND lines on the board, I've noticed that there are jumper wires on the top side of the PCB which are connected to these lanes. Since the power supply PCB is a one-layer PCB, those apparently were needed to complete the routing.

This makes soldering some wires to them really easy since we don't have to mess with the bottom side of the PCB. I've just soldered wires to the respective jumper wires, connected them to the +12V supply, the +3.3V output from the step-down converter and GND, respectively. I'm also using a 2A fuse between the camper's power supply and the Blu-ray player to prevent any dangerous over-current situation in case of a failure or in case I made a mistake.

![Wires soldered to the power-supply jumper wires](/assets/2022/07/bd_connections.jpg)

# Testing

All the new wiring and the step-down converter fit into the original case so I could just put all the stuff inside, replace the 230V cable with the new 12V power cable and connect it to the camper. I'm using [Tesa fabric tape](https://www.amazon.de/gp/product/B00PAC9OAK/){:target="_blank"} to protect the power cable and give it a nicer look.

![Blu-ray player with installed step-down converter and wiring](/assets/2022/07/bd_complete.jpg)

To my surprise, it actually worked on first attempt! The player boots up smoothly, the Blu-ray drive opens and playback works without issues.
