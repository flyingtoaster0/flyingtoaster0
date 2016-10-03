---
layout: post
title:  "Keyboard Prototype 2 - The Breakout Board"
categories: hardware keyboards
---

The keyboard matrix prototype still hasn't arrived, so I figured I would get to work on making the breakout board for the microcontroller.

I felt the need to build this out before designing the keyboard itself so that I can confirm that everything related to the microcontroller is actually connected properly. At this point, I'm actually not totally sure if I'm able to program the chip via USB, or if I need to use the programming pins, so I added a programming header for myself on the board, just in case.

For the curious: here's my schematic for the board:

[![Schematic for the breakout board](/assets/numpad_keyboard/atmega32u4_breakout/atmega32u4_breakout_schematic.png)](/assets/numpad_keyboard/atmega32u4_breakout/atmega32u4_breakout_schematic.png)

A lot of this is simply adapted from [Adafruit's ATmega32u4 breakout board schematic](https://learn.adafruit.com/assets/35683). I felt that even if the board is mostly the same, that it would be good to get the practice laying things out manually.

My first iteration of the board was actually really big and ugly and wouldn't have even fit on a breadboard. So once again, I used Adafruit's breakout board as a guide for pin positioning, and routing. It's a little long, but it ended up alright in the end!

[![ATmega32u4 breakout board render](/assets/numpad_keyboard/atmega32u4_breakout/atmega32u4_breakout_render_front.png)](/assets/numpad_keyboard/atmega32u4_breakout/atmega32u4_breakout_render_front.png)

[![ATmega32u4 breakout board render](/assets/numpad_keyboard/atmega32u4_breakout/atmega32u4_breakout_render_back.png)](/assets/numpad_keyboard/atmega32u4_breakout/atmega32u4_breakout_render_back.png)

I'm pretty happy with the way it turned out! I've sent this board to OSH park as well, but given that the 2x2 matrix prototype hasn't arrived yet, I imagine it will be November by the time I'm soldering this board.

Next time, I solder the matrix board and check to see if my spacing is off.