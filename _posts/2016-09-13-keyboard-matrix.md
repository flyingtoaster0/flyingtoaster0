---
layout: post
title:  "Keyboard Matrix"
categories: hardware keyboards
---

In this post, I will go over digital I/O and keyboard matrices. I'll finish up by showing a proof of concept of what will become the basis for the design of my PCB.

A keyboard is essentially big button matrix. When any one of these buttons is pressed, the state of an input pin on a microcontroller is changed, and then a computer is notified of the keypress event.

# Controlling Pin Logic

I have already been demonstrating detecting the state of an input pin in previous posts, but I'll now go over how it actually works.

A digital input pin is capable of detecting two states: "1" and "0". In digital circuits, supply voltage (often 3.3V or 5.5V) being provided to the an input pin is generally interpreted as "1", while 0V is considered "0".
When an input pin is not connected to anything, it is said to be in the "floating" or high impedance state (also called Hi-Z). While floating, a pin does not have any defined logic value. Although reading its input will result in a "1" or "0", this value will be unrealiable, as the pin is not actively having current flowing towards or away from it. Just because you have nothing connected to an input pin, it doesn't mean that it is a logical zero.

One can eliminate the floating state by using pull-up and pull-down resistors.

## Pull-Up Resistors

Suppose we want to detect when a switch is being pressed. Someone inexperienced with circuits might say "Oh that's easy! I'll connect my input to the source voltage with a switch in between. When it the switch is pressed, the input goes high!"

![alt text](/assets/numpad_keyboard/input_switch_vcc.png)

Our young engineer is correct that the pin will be pulled high when the switch is pressed, but is forgetting that we do not actually know the state of the pin while the switch is open. It *might* be "0", but it also might not be.

One way to fix this problem is with a pull-up resistor:

![alt text](/assets/numpad_keyboard/pullup_resistor.png)

In the circuit above, the input is connected to the VCC with a high-resistance (usually 3.3kΩ or 10kΩ) resistor in between. The input is naturally pulled to "1", and the resistor protects it from receiving too much current. A switch is placed in between the input and GND. In a circuit, current will tend to flow towards the path of least resistance. Because a switch has significantly less resistance than a resistor, when the switch is pressed, current will flow between the input and ground, bringing the input to "0".

When the switch is open, the pin is *pulled up* to VCC.

## Pull-Down Resistors

By reversing the locations of the switch and the resistor, we can create a pull-down resistor. In this case, the state of the pin will be "0" by default.

![alt text](/assets/numpad_keyboard/pulldown_resistor.png)

When the switch is open, the pin is *pulled down* to ground.

## Internal Pull-Up Resistors

Many microcontrollers have pull-up resistors built into their pins. By running the following two lines, a pin is made to be an input, and its internal pull-up resistor is turned on. This means that the pin will now read "1" unless it is somehow pulled to ground by something else in the circuit.

{% highlight c %}

// PB3 is an input, but its state is floating. Its value cannot reliably be read.
DDRB = ~_BV(PB3);

// PB3's internal resistor is turned on. PB3 is pulled to "1" by default.
PORTB = _BV(PB3);

{% endhighlight %}


# The Matrix

Now that we can detect button presses, but we still have a problem: any keyboard is likely to have significantly more keys than its microcontroller has input pins. To support as many keys as we need, we will need to build a keyboard matrix.

In a keyboard matrix, the microcontroller's inputs are arranged in *m* rows, and are connected to *n* columns of outputs. The inputs are by default, pulled to "1" by each one of their internal pull-up resistors. The states of the buttons are checked by cycling through the outputs, and setting one of them to "0" at a time. When a button in an active column is pressed, the input that the button is connected to will be pulled to "0", indicating to the microcontroller that a key was pressed. Pressing a button in a column that is inactive (i.e. its output is "1") will do nothing, because the input is also already "1".

![alt text](/assets/numpad_keyboard/keyboard_matrix_1_row.png)

In the example above, we have a single input with *n* outputs. The outputs will be cycled through, "taking turns" being "0". Given that we have only one input, the key event to fire is determined entirely by the current state of the outputs.

We can also add as many rows as we have available inputs. The logic works as it did before; we cycle through the outputs, but now that we have more rows, we must check the state of every input each time an output is set to "0". 

The result is *m* * *n* possible key events.
 
![alt text](/assets/numpad_keyboard/keyboard_matrix_m_by_n.png)

### Diodes And Ghosting

Ghosting is a term that is used to describe a phenomenon that occurs when a combination of keypresses results in the computer thinking that a different key was pressed altogether. Take a look at the following image:

![alt text](/assets/numpad_keyboard/keyboard_matrix_ghosting.png)

I removed the diodes that were present in the previous circuits. In this picture, three switches are being pressed at the same time. Two of them share row *m*, and two of them share column *2*. Suppose column *1* is currently active. In this case, row *2* is connected to the currently active column *1* because current is allowed to flow "back" through the switch at (Rm, C2). As a result, the keyboard would think that the key at (Rm, C2) was pressed. 

By placing diodes on every switch, we can enforce the direction of the current, and prevent the current from flowing "backwards" through the switches.

# Implementing The Matrix

I wanted to make a small proof of concept for myself before designing a PCB, just to make sure I actually knew what I was doing. The circuit itself is pretty much what you would expect after reading this post. I decided to stick to a 2 * 2 matrix.

![alt text](/assets/numpad_keyboard/keyboard_matrix_2by2_implementation.png)

# The Code

The code itself is also pretty simple. The constants used for keypressed can be found [here](http://www.fourwalledcubicle.com/files/LUFA/Doc/130303/html/group___group___u_s_b_class_h_i_d_common.html). There's likely cleaner way to organize this logic, but that can come later. As long as it's registering keypresses for now, I'm happy.

{% highlight c %}
int main() {
    // ...

    // Set up inputs, turn on pull-up resistors
    DDRD = ~( _BV(PD0) | _BV(PD1) );
    PORTD = ( _BV(PD0) | _BV(PD1) );

    // Set up outputs
    DDRB = ( _BV(PB5) | _BV(PB4) );
	
	// ...
}


// ...


// Checking keypresses
bool CALLBACK_HID_Device_CreateHIDReport(
    USB_ClassInfo_HID_Device_t* const HIDInterfaceInfo,
    uint8_t* const ReportID,
    const uint8_t ReportType,
    void* ReportData,
    uint16_t* const ReportSize)
{
	USB_KeyboardReport_Data_t* KeyboardReport = (USB_KeyboardReport_Data_t*)ReportData;

	uint8_t UsedKeyCodes = 0;

	// Turn off output PB4, turn on all other outputs
	PORTB = ~(_BV(PB4));
	
	// Check (PD0, PB4)
	if ((~PIND & _BV(PD0))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_DOT_AND_DELETE;
	}
	
	// Check (PD1, PB4)
	if ((~PIND & _BV(PD1))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_1_AND_END;
	}
	
	// Turn off output PB5, turn on all other outputs
	PORTB = ~(_BV(PB5));
	
	// Check (PD0, PB5)
	if ((~PIND & _BV(PD0))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_2_AND_DOWN_ARROW;
	}
	
	// Check (PD1, PB5)
	if ((~PIND & _BV(PD1))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_3_AND_PAGE_DOWN;
	}

	*ReportSize = sizeof(USB_KeyboardReport_Data_t);
	return true;
}

{% endhighlight %}

This worked for the most part; most of my keypresses were getting recognized. *Most*. I was getting weird behavior where typing what I expected to be "1" would come out as "3", or pressing a button would instead type "13" together. 

### Debounce

When a switch is pressed, two metal contacts come into contact with each other, closing the switch and allowing current to flow. When a switch is opened again, it has a tendency to bounce closed again momentarily, registering multiple press events.

Debouncing is a a term given to hardware or software that prevents such multiple presses from being registered.

In my case, I think my problem was actually that there wasn't enough time between when an active output changed, and when the rows started being checked. I fixed this by adding a 5ms sleep every time I changed an output.

This solved my problem, and should also double as a debounce mechanism for when mechanical switches are being used. Here's the updated code:

{% highlight c %}

bool CALLBACK_HID_Device_CreateHIDReport(
    USB_ClassInfo_HID_Device_t* const HIDInterfaceInfo,
    uint8_t* const ReportID,
    const uint8_t ReportType,
    void* ReportData,
    uint16_t* const ReportSize)
{
	USB_KeyboardReport_Data_t* KeyboardReport = (USB_KeyboardReport_Data_t*)ReportData;

	uint8_t UsedKeyCodes = 0;

	// Turn off output PB4, turn on all other outputs
	PORTB = ~(_BV(PB4));
	_delay_ms(5);
	
	// Check (PD0, PB4)
	if ((~PIND & _BV(PD0))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_DOT_AND_DELETE;
	}
	
	// Check (PD1, PB4)
	if ((~PIND & _BV(PD1))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_1_AND_END;
	}
	
	// Turn off output PB5, turn on all other outputs
	PORTB = ~(_BV(PB5));
	_delay_ms(5);
	
	// Check (PD0, PB5)
	if ((~PIND & _BV(PD0))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_2_AND_DOWN_ARROW;
	}
	
	// Check (PD1, PB5)
	if ((~PIND & _BV(PD1))) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_3_AND_PAGE_DOWN;
	}

	*ReportSize = sizeof(USB_KeyboardReport_Data_t);
	return true;
}

{% endhighlight %}

And here's my prototype laid out on the breadboard, working as expected!

![alt text](/assets/numpad_keyboard/pressing_four_buttons.gif)