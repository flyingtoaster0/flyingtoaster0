---
layout: post
title:  "Typing Characters"
date:   2016-08-25 21:00:00 -0400
categories: hardware keyboards
---

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

![alt text](/assets/numpad_keyboard/typing_characters/pushing_button.gif "Pushing buttons. Producing zeroes.")

Super excited about this. This is awesome. 

Now it's time to look into how to go about supporting 22 keys with less than 22 input pins. I can imagine a few ways to do this, but I'm sure there's a "correct" way to do this. I'll look into it for next time.