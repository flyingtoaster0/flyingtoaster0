---
layout: post
title:  "The AVR Is A Keyboard"
date:   2016-08-24 21:00:00 -0400
categories: hardware
---

During the day, I had the sudden realization that ASCII characters weren't enough, and that numpads have their own keyboard codes. I would also need support for numlock which obviously isn't an ASCII character. So I did some more looking around, and it seemed that the thing to do was write my own firmware in C with [LUFA](http://www.fourwalledcubicle.com/LUFA.php), and flash the chip directly with [avrdude](http://www.nongnu.org/avrdude/).

So I grabbed the LUFA examples and had a look at the code. Other than setting setting up a development environment in Windows (Thank you [MinGW](http://www.mingw.org/)!), the whole thing was relatively painless. In the end, all I had to do was run `make`, short the Pro Micro's reset and ground pins twice to get it into its bootloader, and then run:

{% highlight bash %}
avrdude.exe -p atmega32u4 -c avr109 -P COM4 -U flash:w:Keyboard.hex
{% endhighlight %}

![alt text](/assets/flashing_pro_micro.gif "Flashing the Pro Micro")

...and it worked! Nothing failed and nothing was bricked; I was impressed!

I was also initally surprised to see that my computer no longer identified my Pro Micro as an Arduino Leonardo, but then it immediately made a huge amount of sense after having thought about it for more than 0.8 seconds.

What was even cooler though was that my computer actually now recognized the Pro Micro as an actual keyboard instead!

![alt text](/assets/device_manager_two_keyboards.png "Two keyboards!")

Holy crap, two keyboards!

That's all for tonight. I feel accomplished.