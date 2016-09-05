---
layout: post
title:  "AVR Ports"
categories: hardware keyboards
---

After getting a pushbutton to send a single key event to the computer, my next goal was to wire up a few more "keys" in the way an actual keyboard circuit would be laid out. I had a lot of trouble getting simple things working. I couldn't even reliably turn on a pin. It didn't take long before I realized that I actually had no idea how all the macros provided by LUFA worked. So I did some reading on how pins on AVR microcontrollers actually work.


# Arduino and the ATmega328p

My first experience with microcontrollers was, like many people, with an Arduino Uno. Arduino is an open source prototyping platform that makes it easy to get started with hardware projects. They provide their own IDE and C-like programming language to program the boards.

*Hello World!* with Arduino or other microcontrollers is generally blinking an LED, because what better way is there to say hello to the world than flashing a bright light in its face? Hello world in Arduino would look like this:


{% highlight c %}
void setup() {
  // Initialize digital pin 13 as an output.
  // Assume our LED's anode is connected to this pin.
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH);
  delay(1000);
  digitalWrite(13, LOW);
  delay(1000);
}
{% endhighlight %}

...And that's it! It's all pretty self-documenting even without comments. It almost reads like English:

{% highlight c %}
Set up our hardware by setting pin 13 to output mode
Go into the loop
Set pin 13 to high - turning the LED on
Wait 1000ms
Set pin 13 to low - turning the LED off
Wait 1000ms
Loop forever. 
{% endhighlight %}
Simple!

After digging around, I found a similar example program in pure C that uses the LUFA framework. This was the first program I flashed to my ATmega32u4:


{% highlight c %}
int main() {
  DDRB = _BV(PB0);

  while(1) {
  PORTB ^= _BV(PB0);
  sleep(1000);
  }
}
{% endhighlight %}

...While I instinctively knew what this program was supposed to do, I still had no idea what `DDRB = _BV(PB0)` or `PORTB ^= _BV(PB0)` were doing. Obviously, line 2 is doing some kind of setup, the loop toggles the LED on and off, but what do all these capital letters mean? Are they supposed to be macros or something?

Short answer: Yes and no.

Long answer:


# AVR Ports

Arduino is great. It takes a lot of the esoteric aspects of microcontroller programming and makes what you're doing very obvious. The problem is that this can give a false understanding of how pins on an AVR work at a hardware level.

I started by looking at pinout for the ATmega32u4:

![alt text](/assets/32U4PinMapping.png "ATmega32u4 pinout")

In my case, `PB0` (Pin 8 on the diagram) seems to be connected to the LED on my Pro Micro. Neat. I tried modifying the code to work with other pins too, connecting an LED to the appropriate pin. `PB1` worked; `PD4` was also good.

In order to detect keypresses on the keyboard, I would need to source voltage from one pin to another. I found some example code for setting a pin as an input:

{% highlight c %}
// Set PB0 as an input, and the rest as outputs
DDRB = ~_BV(PB0);

// Set PB0 and PB1 as inputs, and the rest as outputs
DDRB = ~(_BV(PB0) | _BV(PB1));

// Read the state of PB0
variable = (PINB & _BV(PB0));
{% endhighlight %}

I still wasn't super sure about what was going on, other than that it looked like the code was doing some bitwise logic. Digging around, I found a GitHub repo where the author was doing something similar. Here's an sample of the keypress checking:

{% highlight c %}
// In keypad.h
#define PB_8_BIT 4


// In main.c
#include "keypad.h"

// ...

if (!((1<<PB_8_BIT)&PINB))
  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_8_AND_UP_ARROW;
{% endhighlight %}

So it looks like there's some actual bit shifting going on here. Based on the definition in `keypad.h`, the condition inside the if statement could be written `!((1<<4) & PINB)`. 

Looking back at my own implementation, I was checking the pushbutton's press state with:

{% highlight c %}
!(PINB & _BV(PB0))
{% endhighlight %}

I figured that the number `0` in `PB0` might map to the bit shifting mentioned above, so I changed my code up a bit:

{% highlight c %}
int main() {
  DDRB = 1<<0; // Was DDRB = _BV(PB0);

  while(1) {
  PORTB ^= 1<<0; // Was PORTB ^= _BV(PB0);
  sleep(1000);
  }
}
{% endhighlight %}

...And the LED was still happily blinking away.
After trying this out with a few other values, it became pretty obvious that somewhere in a header file, something like the following line of code must exist:

{% highlight c %}
#define _BV(N) 1<<N
{% endhighlight %}

### After actually looking at the source

I found the definition of `_BV(N)`. It's in `avr/sfr_defs.h`.

{% highlight c %}
 /** \def _BV
       \ingroup avr_sfr
   
       \code #include <avr/io.h>\endcode
   
       Converts a bit number into a byte value.
   
       \note The bit shift is performed by the compiler which then inserts the
       result into the code. Thus, there is no run-time overhead when using
       _BV(). */
       
  #define _BV(bit) (1 << (bit))
{% endhighlight %}

`avr/iom32u4.h` is the file that includes definitions for the ATmega32u4. In it, I found the following definitions:

{% highlight c %}
#define PORTB0 0
#define PORTB1 1
#define PORTB2 2
#define PORTB3 3
// ...and so on
{% endhighlight %}

Makes sense.

# I/O Registers

I found that `DDRx`, `PORTx`, and `PINx` are actually registers in the AVR microcontroller that are actually very essential to doing anything with an AVR.

In Arduino, one can manipulate the pins directly (e.g. pin 13), but when programming an AVR in C, it is more accurate to think of pins as being *a part of a single port*. 

For example, as an input/output pin, `PB3` is better referred to as "pin 3 of port B on the AVR" rather than "pin 11 of the AVR".

### Data Direction Register

`DDRx` is the Data Direction Register on the AVR, where `x` is the port to be set (A, B, C, etc.)

Writing a `1` to a pin of a port will set that pin to be an output, while writing a `0` will set that pin as an input. Note that pins on a port can be configured independently of each other.

Setting `PB3` to be an input, and the rest of port B's pins to be outputs could look like this:

{% highlight c %}
DDRB = _BV(PB3);
{% endhighlight %}

`~_BV(PB3)` is equal to `0b11111011`, making the 3rd pin on port B an input and the rest of the pins outputs.

### Port Data Register

`PORTx` is the Port Data Register, where once again, `x` is the letter corresponding to the port you're setting.

#### Outputs

For a pin configured as an output, writing `1` will set that pin to a high state; writing `0` will set that pin to low.

Setting `PB3` as an output and then setting its state to high could look like this:

{% highlight c %}
// Set PB3 to be an output, the rest of port B's pins are inputs
DDRB = _BV(PB3);

// Set PB3 to high
PORTB = _BV(PB3);
{% endhighlight %}

`_BV(PB3)` is equal to `0b00000100` in both cases, making the 3rd pin on port B an output, and set to high.

#### Inputs

For a pin configured as an input, setting `1` will enable the pin's internal pull-up resistor. Writing `0` will disable it.

Setting `PB3` as an input and then turning on its internal pullup resistor could look like this:

{% highlight c %}
// Set PB3 as an input, the rest of port B's pins are outputs
DDRB = ~_BV(PB3);

// Turn on PB3's pull-up resistor
PORTB = _BV(PB3);
{% endhighlight %}

`~_BV(PB3)` is equal to `0b11111011`, making the 3rd pin on port B an input, and turning on its pull-up resistor.

### Port Input Pins Register

`PINx` is the Port Input Pins Register.
This register is used to read the current state of input pins on port `x`.

Suppose `PB3` and `PB5` are both set to high, and the rest of port B's pins are all low. Then `PINB` is equal to `0b00010100`.

# Putting it all together

Let's take all we've learned today and put it into a tangible example.

For example, suppose I wanted to set up pins 3 and 4 on `PORTB` as inputs and then check the state of pin 3. An excerpt from a program doing this could look like the following:

{% highlight c %}
// Strictly speaking, PB3 AND PB4 are already inputs by default
// This also sets all other pins on PORTB to outputs
DDRB = ~(_BV(PB3) & _BV(PB4));

// Turn on the pull-up resistors on PB3 and PB4, turn off all other outputs
PORTB = _BV(PB3) | _BV(PB4);

// Check the state of pin 3 on PORTB
uint8_t stateOfPinB = (PINB & _BV(PB3));
{% endhighlight %}

Going through each of the statements, we can see the bitwise operations that are happening:

{% highlight c %}
DDRB = ~(_BV(PB3) & _BV(PB4));

_BV(PB3) = 1<<3 = 00000100
_BV(PB4) = 1<<4 = 00001000

  00000100
& 00001000
------------
~ 00001100
------------
  11110011

DDRB = 0b11110011
{% endhighlight %}

Setting outputs and `PB3` and `PB4`'s pull-up resistors:

{% highlight c %}
PORTB = _BV(PB3) | _BV(PB4);

_BV(PB3) = 1<<3 = 00000100
_BV(PB4) = 1<<4 = 00001000

  00000100
| 00001000
------------
  00001100

PORTB = 0b00001100
{% endhighlight %}

Suppose the states of both pins were high. The very simple bitwise arithmetic we did while checking the state is:

{% highlight c %}
stateOfPinB = (PINB & _BV(PB3));

PINB = 00001100
_BV(PB3)) = 00000100

  00001100
& 00000100
------------
  00000100
  
stateOfPinB = 0b00000100
{% endhighlight %}

which is of course `8` in base 10, and evaluates to `true` as a `bool` in C.

# Oh hai world

With what we know now, it's very easy to decipher the *Hello World!* example I showed earlier. Here it is again:

{% highlight c %}
int main() {
  DDRB = _BV(PB0);

  while(1) {
  PORTB ^= _BV(PB0);
  sleep(1000);
  }
}
{% endhighlight %}

Set pin 1 of port B as an output.

The rest of the pins on port B are inputs.
{% highlight c %}
DDRB = _BV(PB0);

  00000001
<<       0  (Shift 0)
------------
  00000001  (New value of DDRB)
{% endhighlight %}

Reverse the state of pin 0 on port B

First pass of the loop:
{% highlight c %}
PORTB ^= _BV(PB0);

  00000000  (Value of PORTB)
^ 00000001  (_BV(PB0))
------------
  00000001  (New value of PORTB)
{% endhighlight %}

Reverse the state of pin 0 on port B

Second pass of the loop:
{% highlight c %}
PORTB ^= _BV(PB0);

  00000001 (PORTB)
^ 00000001 (_BV(PB0))
------------
  00000000 (New value of PORTB)
{% endhighlight %}

...And repeat for eternity.

# Appendix
## There are 10 types of people...

This post requires basic knowledge of bitwise operators. I figured I would dump some simple explanations and examples here for sake of completeness.

### OR

Two bits are compared. If either one of them is `1`, then the result is `1`. Otherwise, the result is `0`.

In C, it is represented by `|`.

e.g.

```
  00000100
| 00010000
------------
  00010100
```

### AND

Two bits are compared. If both of them are `1`, then the result is `1`. Otherwise, the result is `0`.

In C, it is represented by `&`.

e.g.

```
  00000101
& 00010100
------------
  00000100
```

### NOT

All bits in a byte are reversed. `0`s become `1`s, and `1`s become `0`s.

In C, it is represented by `~`.

e.g.

```
~ 00000101
------------
  11111010
```

### XOR

Exclusive OR. Two bits are compared. If they are different, then the result is `1`. If they are the same, the result is `0`.

In C, it is represented by `^`.

e.g.

```
  00110010
^ 10111011
------------
  10001001
```

### Left Shift

Shift the bits to the left the specified number of times.

In C, it is represented by `<<`.

e.g.

```
1<<3

  00000001
<<       3  (3 times)
------------
  00001000
```

### Right Shift

Shift the bits to the right the specified number of times.

In C, it is represented by `>>`.

e.g.

```
8>>3

  00001000
>>       3  (3 times)
------------
  00000001
```