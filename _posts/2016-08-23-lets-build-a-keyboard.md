---
layout: post
title:  "Let's Build A Keyboard!"
date:   2016-08-23 21:00:00 -0400
categories: hardware keyboards
---

While browsing Massdrop, I came across a cool looking numpad and almost joined the drop, but decided to look up some reviews on it. I found a thread of geekhack where the poster asked about it, and people started comparing it to other kits. While looking at the kits, I came across [this thread](https://geekhack.org/index.php?topic=35894.0) where a forum member describes a numpad he build from scratch. I looked at the post for a bit and eventually had an "I could do that..." thought. So I started doing research.

I started by looking at what microcontroller to use, and after browsing [this very resourceful website](https://www.google.ca/?ion=1&espv=2#q=how%20to%20build%20a%20keyboard) I found that a lot of people were using the ATmega32U4 for building custom keyboards. I then remembered that I had a [Pro Micro](https://www.sparkfun.com/products/12640) kicking around, so I pulled that out and fired up the Arduino IDE. 

![alt text](/assets/numpad_keyboard/lets_build_a_keyboard/pro_micro.jpg "A 5V/16MHz Pro Micro")

I learned that the Arduino library has some pretty basic keyboard commands. The most useful one looks like this:

{% highlight c %}
Keyboard.write(char);
{% endhighlight %}

It pretty much just sends an ASCII character to the computer. I compiled, and it worked! Cool, that's enough for tonight.
