---
title: "Acrylic Sandwich Case"
layout: post
categories: hardware keyboards
---

There are a few ways to construct a case for a keyboard.

One way is out of plastic using a mold. Another common method is to stack pieces of some material on top of each other. This is sometimes called a sandwich case.

Given that I don't have immediate access to 3D printing or manufacturing equipment, I chose to go with the second method. For sandwich-style keyboard cases, acrylic is often used. I found a local business that does small scale laser cutting so I went with them in the end.

To make the designs for the cuts, I used two online tools. 

### Keyboard Layout Editor

[http://www.keyboard-layout-editor.com/](http://www.keyboard-layout-editor.com/)

This tool is great for defining custom keyboard layouts. In retrospect, I really should have done this before even designing the PCB. I honestly don't know if the PCB will even fit with the layout I chose, but oh well, I'll find out soon enough.

### Plate & Case Builder

[http://builder.swillkb.com/](http://builder.swillkb.com/)

This tool takes the output from Keyboard Layout Editor to lay out where to place holes for the switches. You can tweak various parameters like switch type, edge radius, drill holes, etc. The Plate & Case Builder outputs a handful of .svg files which define the various layers of the case.

### Getting it all cut

To get the case manufactured, I found a local business called [Hot Pop Factory](http://www.hotpopfactory.com/) that laser cuts acrylic. After a few days, I went to pick up my case.

![alt text](/assets/numpad_keyboard/acrylic_case/numpad_case_empty.png)

I started plugging in the switches and was please to find that they all fit perfectly! I used Cherry MX clear switches for the top row (arbitrary function keys), and blue switches for the rest. I like the clicky feel when typing numbers.

![alt text](/assets/numpad_keyboard/acrylic_case/case_with_switches.jpg)


### Stabilizers

![alt text](/assets/numpad_keyboard/acrylic_case/switch_with_springs.jpg)

After all the keys were in place, I started assembling the key stabilizers. Larger keys like Spacebar, Enter, Numpad 0, or Numpad + require stabilizer springs to prevent the keys from wobbling if you don't press down in the center. 

A key stablizer consists of three parts:
1. Plastic inserts that fit into the plate
2. The spring
3. Keycap inserts that hook into the spring

The spring fits into the plastic inserts, and then the keycap inserts hook into the sides of the spring. Together, this keeps larger keys relatively level when being pressed off centre.

I set up the spring on the switch for "0" first; it fit nicely.

![alt text](/assets/numpad_keyboard/acrylic_case/case_springs_good_1.gif)

The springs for the "Enter" and "+" keys, however, did not feel quite right. After putting the springs on, the inserts seemed loose in the plate, and the springs felt like they had a constant tension being applied to them. If you compare the gifs above and below, you can sort of see what I mean.

![alt text](/assets/numpad_keyboard/acrylic_case/case_springs_bad.gif)

I opened my plate design and found that the holes for the stabilizer inserts are actually not symmetrical. This means that there was slightly less room on the spring's side, hence the constant tension. Below in red, we have the "bad" cut of the stablilizer inserts; the fixed cut is in blue.

![alt text](/assets/numpad_keyboard/acrylic_case/stabilizer_cut_comparison.png)

I fixed my design and sent it over to have it re-cut. I picked up the new cut after a few days and was very pleased with the way everything fit.

![alt text](/assets/numpad_keyboard/acrylic_case/case_springs_good_2.gif)

The next step is to wait for the PCB to arrive to see if everything fits. I'll update when it arrives.