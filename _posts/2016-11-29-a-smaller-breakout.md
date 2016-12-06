---
layout: post
title:  "A Smaller Breakout Board"
categories: hardware keyboards
---

After the failure of the breakout board I designed, I had to decide what to do next. I found a site called macrofab.com that provides PCB fabrication and assembly services. They source their components from digikey, so I would be able to shop for parts there and specify them in my order on macrofab. 

Later that night, I spoke to a friend of mine who is an electrical engineer and explained my problem. He asked why I didn't just use a breadboard to test my circuit. I told him that I couln't because the microcontroller I was using only comes in a suface mount package. He then told me that I should just get a board that converts it to a through hole package that could be used in a breadboard, and then just wire it up. Not sure why I didn't think of that myself!

I fired up KiCad and started designing, but after a few minutes, I figured that what I was designing may already exist.
It did.

[Pic of empty board w/o micro]

This time, I was very careful to not fry the microcontroller when melting the solder paste on the stove. To find the minimum temperature the solder paste would melt, I put a drop of paste on the frying pan, turned up the heat until it melted, turned it back down until it solidified, and then turned it up until it melted again. This was the termperature to which I would preheat the frying pan for soldering.

I applied the paste the the pads, lined up the microcontroller, and placed it on the stove. The paste melted in just a few seconds, and with no solder bridges between any of the pins. Perfect!

[pic of board w/micro]

Next, I soldered the through hole pins. To solder through hole pins, I like to stick all my pins in a breadboard, and then put whatever I'm soldering on top of them. This ensures the pins will be nice and straight.

[pic of micro on top of pins]

Next, I followed the schematic I designed for the breakout board I tried to assemble previously. After everything was connected, I plugged it in to my computer and got a "new device found" notification!

[pic?]















I neglected to make a post about the keyboard matrix, but it turned out great; I'll show it in the next post.

The breakout board PCB arrived, and it looked pretty great!

![Unassembled breakout](/assets/numpad_keyboard/assembling_the_breakout/atmega32u4_breakout_empty.jpg)

Given that I'm using surface mount components, I had to find a way to solder them. Given their tiny pin size, I felt that soldering with an iron wouldn't work as well. 

I found various sources stating that one can solder surface mount components using solder paste and a skillet. 

![A syringe of solder paste](/assets/numpad_keyboard/assembling_the_breakout/solderpaste_syringe.jpg)

As for the skillet, I used an old frying pan.

![The soldering skillet of science!](/assets/numpad_keyboard/assembling_the_breakout/soldering_skillet_of_science.jpg)

All it took was to smear some of the stuff on the pads, and then heat it up on the stove. It's cool to see it suddenly melt into a liquid; the properties of liquid metal make it all suck together when it melts.

![Before the paste melted](/assets/numpad_keyboard/assembling_the_breakout/frying_pan_paste.jpg)

![After the paste melted](/assets/numpad_keyboard/assembling_the_breakout/frying_pan_paste_melted.jpg)

I was really happy how that test run went, so I went to solder the microcontroller. The problem with the microcontroller was that it has a bunch of tiny pins beside each other. Really tiny pins. They're smaller than the diameter of the tip of the syringe I was using for paste, so it was impossible to apply it to the pads in appropriate amounts.

![Large syringe end](/assets/numpad_keyboard/assembling_the_breakout/big_syringe_hole.jpg)

I found a technique for applying solder to lots of tiny pins where you smear solder across the entire row with a toothpick. Because liquid metal tends to adhere to other metal, it'll all get sucked up onto the pads; it worked.

I then went to put the USB connector on, and I found that it didn't fit as I expected. The connector itself has little oval shaped feet that are supposed to secure it to the board. For some reason, they didn't fit.

I checked my design in KiCad, and compared it to what OSH Park had rendered from my design. The left is a render from my design in KiCad, and the right is what OSH Park read from the design.

![Oval/circle hole comparison](/assets/numpad_keyboard/assembling_the_breakout/oval_circle_holes.png)

Crap. 
I later found out that OSH Park actually specifies that their fab can't do oval-shaped holes. What I did instead is bend back the rear legs so that the pins actually made contact with the pads on the board. After a few attempts, it kind of worked.

After the USB connector was attached, it was just a matter of getting the rest of the components on. The board looked quite nice in the end. On the right, you can see the legs I bent backwards on the USB connector.

![The fully assembled board](/assets/numpad_keyboard/assembling_the_breakout/atmega32u4_breakout_assembled.jpg)

I plugged it into my computer, and...
It didn't work. I was hoping for a "new device found" message, but nothing. The power LED did turn on, but even that flickered if I didn't hold the USB cable in the right way.

I considered this a failure, but had a few takeaways.

1. The footprint that I used for the USB connector didn't fit because I didn't notice that OSH Park doesn't drill oval holes. Since I plan on continuing to order from OSH Park, I will use a surface mount receptacle next time.
2. The microcontroller didn't work at all when I plugged it in. I'm almost certain that I literally fried it by having it on the stove for too long. Next time, I'll solder all my components at the same time.
3. It is also likely that the stove was just too hot. Next time, I'll test the temperature first by seeing at what stove temperature the solder paste melts and freezes.

All this, of course, was primarily to test the hookup of the microcontroller itself to ensure that it would work when hooked up to a keyboard. At this point, I was considering just having the breakout board fabricated and assembled. In my next post, I'll show what I decided.