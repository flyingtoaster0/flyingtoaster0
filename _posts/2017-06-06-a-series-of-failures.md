---
layout: post
title:  "A Series Of Failures"
categories: hardware keyboards
---

Since December, a lot has happened with regards to the keyboard. Mostly bad things. I'll do my best to summarize everything here.

## Assembling The Keyboard

First up:

The PCB arrived, and it looked beautiful.

![Keyboard PCB](/assets/numpad_keyboard/a_series_of_failures/board1.jpg)

Given that this board contains a lot of tiny components, some of which I cannot solder by hand, I decided to order a PCB stencil. The stencil would allow me to apply solder paste to the PCB's pads accurately. I could then cook the board on the stove to melt the solder and set the components.

Here's what the stencil looked like.

![Stencil](/assets/numpad_keyboard/a_series_of_failures/stencil1.jpg)

The next step was aligning the stencil to the board's pads, and securing it. I used a very accurate method called "taping them together".

![Stencil](/assets/numpad_keyboard/a_series_of_failures/stencil2.jpg)

Then the next step was just smearing solder paste on the holes in the stencil.

![Stencil](/assets/numpad_keyboard/a_series_of_failures/stencil3.jpg)

One thing I failed to foresee is that a frying pan, unlike a skillet, is not flat; this means that the board was heated unevenly. I had to move the board around a little bit in order to get all the paste to melt, but in the end, it looked alright.

![Assembled PCB](/assets/numpad_keyboard/a_series_of_failures/stencil_board_assembled.jpg)

Looks nice, but did not work at all. When I plugged it in, nothing. Inspecting the board more closely, I found that there were a bunch of shorts on the microcontroller, so I heated up my iron, and tried to clean things up. After a certain amount of heating, soldering, and solder removing, PCB pads tend to strip off, rendering the board useless. Luckily these boards ship in orders of 3.

It was a lot of soldering and desolding components, desperately trying to find out where the problem was. At one point, I even got rid of some surface mount components, and soldered through hole components onto the board, just in case.

I ended up with this monster:

![PCB monster](/assets/numpad_keyboard/a_series_of_failures/pcb_monster.jpg)

Eventually I managed to get a board to be recognized by my computer, but it had other problems:

![Smoking microcontroller](/assets/numpad_keyboard/a_series_of_failures/smoking_micro.gif)

In the end, I'm pretty sure I fried about four microcontrollers from shorts or heating them too much when trying to fix shorts. Assembling the board myself wasn't working out.

## Another Way

It became apparent to me that a lot of my problems were brought on by simply not owning professional soldering equipment. Rather than buying some, it figured I would just have the board fabricated and assembled for me.

I found a service called MacroFab which does exactly what I wanted. They even source their parts from Digi-Key, so I went with them.

After about a month and a half, I got my board. I checked the connections, and they all looked great! Even the tiny ones.

There was just one problem with this board.

![Red PCB no holes](/assets/numpad_keyboard/a_series_of_failures/red_pcb.jpg)

Where the hell are all the holes?! 

All those copper circles are supposed to have holes in them. This means that even if the board *did* work, that I wouldn't be able to put switches on it.

This also meant that this board had no vias. Vias are really small holes in the board that allow traces to go from one side of the board to the other. Because this board was missing all copper drill holes, there were no vias, which meant that I couldn't even test to see if the my computer would recognize the board to begin with.

Useless.

The problem was actually in how I submitted my design files to this MacroFab, so I fixed that up, and sent them over.

After another month and a bit, I got a new board. I tried plugging it in... nothing. Again, in another port... nothing.

Hm. Maybe I'll try a different USB cable.

![Red PCB broken](/assets/numpad_keyboard/a_series_of_failures/red_pcb_broken.jpg)

There are no words.

The USB plug came right off the board. Luckily, MacroFab was really nice about the refund.

## Lessons

Six months later, and I still don't have a working keyboard. On the plus side though, there were some lessons learned through this.

### ***I don't own professional soldering equipment***

This made it really hard to even get components soldered in the first place. Even if my PCB was designed properly, I was very likely to mess something up even by just assembling it.

### ***A month and a half is too long to wait for a prototype***

Once I decided that I wouldn't assemble the board myself, I tried ordering it fully assembled. If the turnaround time could have been reduced to two or three weeks, this would have actually been the best option. Waiting over six weeks just to check my work is too much though.

## Next Steps

I'm not done yet. I'm doing to try a different approach that should address the two above problems.