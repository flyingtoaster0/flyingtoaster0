---
layout: post
title:  "Tim Builds A Keyboard"
date:   2016-08-26 12:07:49 -0400
categories: hardware
---

# August 25th, 2016

Today I wanted to get characters actually typing on on the screen.

The keyboard example had a bunch of code that seemed to be doing generic USB stuff that I don't understand. In main(), there was a call to `SetupHardware()` which did pretty much what you would expect it to do - set up pins, etc.

{% highlight c %}
void SetupHardware()
{
#if (ARCH == ARCH_AVR8)
	/* Disable watchdog if enabled by bootloader/fuses */
	MCUSR &= ~(1 << WDRF);
	wdt_disable();

	/* Disable clock division */
	clock_prescale_set(clock_div_1);
#elif (ARCH == ARCH_XMEGA)
	/* Start the PLL to multiply the 2MHz RC oscillator to 32MHz and switch the CPU core to run from it */
	XMEGACLK_StartPLL(CLOCK_SRC_INT_RC2MHZ, 2000000, F_CPU);
	XMEGACLK_SetCPUClockSource(CLOCK_SRC_PLL);

	/* Start the 32MHz internal RC oscillator and start the DFLL to increase it to 48MHz using the USB SOF as a reference */
	XMEGACLK_StartInternalOscillator(CLOCK_SRC_INT_RC32MHZ);
	XMEGACLK_StartDFLL(CLOCK_SRC_INT_RC32MHZ, DFLL_REF_INT_USBSOF, F_USB);

	PMIC.CTRL = PMIC_LOLVLEN_bm | PMIC_MEDLVLEN_bm | PMIC_HILVLEN_bm;
#endif

	/* Hardware Initialization */
	Joystick_Init();
	LEDs_Init();
	Buttons_Init();
	USB_Init();
}
{% endhighlight %}

Pretty much some architechture-related stuff I could delete as well as some calls to some init functions. Looks like this demo was written with some kind of joystick hardware in mind.

A bit further down, I saw this function:

{% highlight c %}
bool CALLBACK_HID_Device_CreateHIDReport(
    USB_ClassInfo_HID_Device_t* const HIDInterfaceInfo,
    uint8_t* const ReportID,
    const uint8_t ReportType,
    void* ReportData,
    uint16_t* const ReportSize)
{
	USB_KeyboardReport_Data_t* KeyboardReport = (USB_KeyboardReport_Data_t*)ReportData;

	uint8_t JoyStatus_LCL    = Joystick_GetStatus();
	uint8_t ButtonStatus_LCL = Buttons_GetStatus();

	uint8_t UsedKeyCodes = 0;

	if (JoyStatus_LCL & JOY_UP)
	  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_A;
	else if (JoyStatus_LCL & JOY_DOWN)
	  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_B;

	if (JoyStatus_LCL & JOY_LEFT)
	  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_C;
	else if (JoyStatus_LCL & JOY_RIGHT)
	  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_D;

	if (JoyStatus_LCL & JOY_PRESS)
	  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_E;

	if (ButtonStatus_LCL & BUTTONS_BUTTON1)
	  KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_F;

	if (UsedKeyCodes)
	  KeyboardReport->Modifier = HID_KEYBOARD_MODIFIER_LEFTSHIFT;

	*ReportSize = sizeof(USB_KeyboardReport_Data_t);
	return false;
}
{% endhighlight %}

Gotcha~ This is where the keypresses are checked and keyboard codes are sent to the computer. Together, these two functions look pretty analogous to Arduino's `setup()` and `loop()` functions - at least for what I need them for.

My goal at this point was to be able to type a character on the screen by pushing a button. Looking around, I found that I could initialize pin PD0 (labelled "3") as an input and make use of its internal pulldown resistor to with these two lines:

{% highlight c %}
DDRB &= ~_BV(PD0);
PORTD |= _BV(PD0);
{% endhighlight %}

I then found a way to check the state of the pin. For the purposes of just *Making it work™*, this was sufficient.

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
	bool is_pressed = !(PIND & _BV(PD0));
	
	if (is_pressed) {
		KeyboardReport->KeyCode[UsedKeyCodes++] = HID_KEYBOARD_SC_KEYPAD_0_AND_INSERT;
	}
	
	*ReportSize = sizeof(USB_KeyboardReport_Data_t);
	return true;
}
{% endhighlight %}

After flashing the chip again, this was the result!

![alt text](/assets/pushing_button.gif "Pushing buttons. Producing zeroes.")

Super excited about this. This is awesome. 

Now it's time to look into how to go about supporting 22 keys with less than 22 input pins. I can imagine a few ways to do this, but I'm sure there's a "correct" way to do this. I'll look into it for next time.

# August 24th, 2016

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

# August 23rd, 2016

While browsing Massdrop, I came across a cool looking numpad and almost joined the drop, but decided to look up some reviews on it. I found a thread of geekhack where the poster asked about it, and people started comparing it to other kits. While looking at the kits, I came across [this thread](https://geekhack.org/index.php?topic=35894.0) where a forum member describes a numpad he build from scratch. I looked at the post for a bit and eventually had an "I could do that..." thought. So I started doing research.

I started by looking at what microcontroller to use, and after browsing [this very resourceful website](https://www.google.ca/?ion=1&espv=2#q=how%20to%20build%20a%20keyboard) I found that a lot of people were using the ATmega32U4 for building custom keyboards. I then remembered that I had a [Pro Micro](https://www.sparkfun.com/products/12640) kicking around, so I pulled that you and fired up the Arduino IDE. I learned that the Arduino library has some pretty basic keyboard commands. The most useful one looks like this:

{% highlight c %}
Keyboard.write(char);
{% endhighlight %}

It pretty much just sends an ASCII character to the computer. I compiled, and it worked! Cool, that's enough for tonight.
