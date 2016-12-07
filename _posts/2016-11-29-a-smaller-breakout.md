---
layout: post
title:  "A Smaller Breakout Board"
categories: hardware keyboards
---

After the failure of the breakout board I designed, I had to decide what to do next. I found a site called macrofab.com that provides PCB fabrication and assembly services. They source their components from digikey, so I would be able to shop for parts there and specify them in my order on macrofab. 

Later that night, I spoke to a friend of mine who is an electrical engineer and explained my problem. He asked why I didn't just use a breadboard to test my circuit. I told him that I couln't because the microcontroller I was using only comes in a suface mount package. He then told me that I should just get a board that converts it to a through hole package that could be used in a breadboard, and then just wire it up. Not sure why I didn't think of that myself!

I fired up KiCad and started designing, but after a few minutes, I figured that what I was designing may already exist.
It did.

![Unassembled breakout](/assets/numpad_keyboard/a_smaller_breakout/atmega32u4_breakout_empty.jpg)

This time, I was very careful to not fry the microcontroller when melting the solder paste on the stove. To find the minimum temperature the solder paste would melt, I put a drop of paste on the frying pan, turned up the heat until it melted, turned it back down until it solidified, and then turned it up until it melted again. This was the termperature to which I would preheat the frying pan for soldering.

I applied the paste the the pads, lined up the microcontroller, and placed it on the stove. The paste melted in just a few seconds, and with no solder bridges between any of the pins. Perfect!

![Assembled breakout](/assets/numpad_keyboard/a_smaller_breakout/atmega32u4_breakout.jpg)

Next, I soldered the through hole pins. To solder through hole pins, I like to stick all my pins in a breadboard, and then put whatever I'm soldering on top of them. This ensures the pins will be nice and straight.

![Soldering the through hole pins using a breadboard](/assets/numpad_keyboard/a_smaller_breakout/atmega32u4_breadboard.jpg)

Next, I followed the schematic I designed for the breakout board I tried to assemble previously. After everything was connected, I plugged it in to my computer and got a "new device found" notification!

After I found the driver for the Atmega32u4's bootloader, I downloaded FLIP which is a program that can be used to flash Atmel microcontrollers that have their stock bootloader. I flashed a .hex file from my keyboard project, and it seemed to flash successfully. I unplugged it, and wired up my 2x2 keyboard matrix. It's a disgusting mess of wires, but it worked!

![Keyboard circuit](/assets/numpad_keyboard/a_smaller_breakout/atmega32u4_breakout_breadboard_keyboard.jpg)

I was able to type without any problems, and numlock worked too! In the gif below, I toggle numlock using small keypad, and then using the numlock key on my actual keyboard.

![Pressing numlock](/assets/numpad_keyboard/a_smaller_breakout/atmega32u4_breakout.gif)

Since this circuit was built from the schematic that I designed for the breakout, I am now confident enough to go ahead and design the board for the numpad itself. More on that next time!
