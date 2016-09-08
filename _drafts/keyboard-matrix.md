---
layout: post
title:  "Keyboard Matrix"
categories: hardware keyboards
---

In this post, I will go over digital I/O and keyboard matrices. I'll finish up by showing a proof of concept of what will become the basis for the design of my PCB.

A keyboard is essentially a device on which there are a lot of buttons. When any one of these buttons is pressed, the state of an input pin on a microcontroller is changed, and then a computer is notified of the keypress event.

- describe "sending" 1 or 0
# Pin Logic




Before we get into 


- Describe pull up/pull down resistors and how they cause the voltage in the pin to be high by default

# Controlling Pin Logic

I have already been demonstrating detecting the state of an input pin in previous posts, but I'll now go over how it actually works.

A digital input pin is capable of detecting two states: "1" and "0". In digital circuits, supply voltage (often 3.3V or 5.5V) being provided to the an input pin is generally interpreted as "1", while 0V is considered "0".
When an input pin is not connected to anything, it is said to be in the "floating" or high impedance state (Hi-Z). While floating, a pin does not have any defined logic value. Although reading its input will result in a "1" or "0", this value will be unrealiable, as the pin is not actively having current flowing towards or away from it. Just because you have nothing connected to an input pin, it doesn't mean that it is a logical zero.

Two ways to eliminate the floating state isare the use of pull-up and pull-down resistors.

## Pull-Up Resistors

Suppose we want to detect when a switch is being pressed. Someone inexperienced with circuits might say "Oh that's easy! I'll connect my input to the source voltage with a switch in between. When it the switch is pressed, the input goes high!"

// image here

Our young engineer is correct that the pin will be pulled high when the switch is pressed, but is forgetting that we do not actually know the state of the pin while the switch is open. It *might* be "0", but it also might not be.

One way to fix this problem is with a pull-up resistor:

// image

In the circuit above, the input is connected to the VCC with a high-resistance (usually 3.3kΩ or 10kΩ) resistor in between. The input is naturally pulled to "1", and is protected from the current by the resistor. Between the input and VCC, a switch is connected to ground. In a circuit, current will tend to flow towards the path of least resistance. Because a switch has significantly less resistance than a resistor, when the switch is pressed, current will flow between the input and ground, bringing the input to "0".

// Why is it called a pull-up resistor

## Pull-Down Resistors

// description
// image
// Why is it called a pull-down resistor


## Internal Pull-Up Resistors

- Describe a 1*n matrix, switch between outputs

- describe the m*n matrix

- describe my implementation

- show my code