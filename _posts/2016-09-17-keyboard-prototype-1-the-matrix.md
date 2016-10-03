---
layout: post
title: "Keyboard Prototype 1 - The Matrix"
categories: hardware keyboards
---


Realizing that I have never designed a PCB myself, I figured I should start small in order to make sure that what I was doing would actually work. I figured that it would be better (and cheaper) to find that my measurements were way off on a small board, rather than realizing that I did everything wrong later.

I decided to start by designing a 2x2 keyboard matrix that would serve as a way for me to check things like measurements, drill hole sizes, and spacing.

I fired up [KiCad](http://kicad-pcb.org/) and got to work on a schematic.

![Schematic for the 2x2 matrix](/assets/numpad_keyboard/2x2_matrix/2x2_matrix_schematic.png)

It looks pretty much the same as the keyboard matrix schematics shown in the previous post. I also added an LED just to check where it should go in conjunction with a switch - just in case I ever wanted to add one.

Once I completed the schematic, I laid out my components. On the board, I plan to place 6 pin headers: one for each row, one for each column, and two for the LED. A couple of hours later, I came up with this:

![PCB design for the 2x2 matrix](/assets/numpad_keyboard/2x2_matrix/2x2_pcb_layout.png)

Here are some snazzy renders of what the board should look like after fabrication.

![2x2 matrix board render](/assets/numpad_keyboard/2x2_matrix/2x2_render_front.png)

![2x2 matrix board render](/assets/numpad_keyboard/2x2_matrix/2x2_render_back.png)

[OSH park](https://oshpark.com/) has pretty good rates for small runs of PCBs. They estimate about a week and a half before I get my board in the mail, so I'll update then.

In the meantime, I intend to make a prototype for the part of the keyboard that contains the microcontroller. It requires a few more components than the matrix does in order to function, so I feel like that's a suitable next step. 

It'll probably just end up looking like a [breakout board](https://programmingelectronics.com/what-is-a-breakout-board-for-arduino/) for the ATmega32u4 anyway.

See you next time!