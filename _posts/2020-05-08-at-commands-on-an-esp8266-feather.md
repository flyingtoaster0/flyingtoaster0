---
layout: post
title:  "AT Commands on an Adafruit ESP8266 Feather"
categories: hardware microcontrollers
---

# The ESP8266

The ESP8266 is a WiFi module made by Espressif. It is often added to projects to cheaply add WiFi capabilities.

# Adafruit Feather HUZZAH with ESP8266

Adafruit manufactures an ESP8266 breakout board as a part of their Feather ecosystem. Feathers are a series of development boards designed in the same form factor, designed to be stackable and modular.

I happened to have an ESP8266 Feather, and wanted to see if I could get another device to communicate to it via AT commands.

# AT Commands

The ESP8266 board generally ships with firmware that responds to commands sent over a serial connection. Among other things, these commands allow connecting to a WiFi network, and making network calls.

AT commands are described at length [here](https://www.espressif.com/sites/default/files/documentation/4b-esp8266_at_command_examples_en.pdf) and [here](https://www.itead.cc/wiki/ESP8266_Serial_WIFI_Module#AT_Commands).


# Flashing the AT firmware

## Getting the tools

Although the firmware containing the AT commands typically come on ESP8266 microchips, the ESP8266 feather from adafruit will not. This means we will need to flash the AT firmware to the chip.

Note that the following was made functional by 10% understanding, and 90% experimentation.

Go [here](https://www.espressif.com/en/products/socs/esp8266ex/resources). We're going to get the tool to flash the firmware, as well as the firmware itself.

Under "Tools", download the "Flash Download Tools". This is the tool we'll use to flash the firmware. At the time of writing, it seems like this is only available for Windows.

Under "AT", select the "ESP 8266 AT NonOS Bin V1.7.3" firmware.

Extract the contents of both archives, and run the flash download tool. 

Choose "Developer Mode"

Choose "ESP8266 DownloadTool" (Not a typo)

## Flashing the firmware

Contained in the firmware zip is a few files. These are binaries that need to be flashed to various memory addresses on the ESP8266.

Looking at a README found in the zip with the binaries, I found the addresses where each .bin needs to be flashed. See /bin/at/README.md.

```
Flash size 16Mbit-C1: 1024KB+1024KB
boot_v1.2+.bin              0x00000
user1.2048.new.5.bin        0x01000
esp_init_data_default.bin   0x1fc000
blank.bin                   0xfe000 & 0x1fe000
```

This setting was chosen based on experimentation, despite the fact that the chip apparently has 32Mbit of memory.

Here are the settings used in the ESP8266 download tool:

![ESP8266 Download Tool Settings](/assets/esp8266_feather/download_tool_settings.png)

Use the COM port on your own machine. This can be found in the Device Manager under COM ports. Find it by unplugging and re-plugging the Feather.

# Using AT Commands

I've had success running AT commands in two ways that will be outlined here. In both cases, I had the Feather wired up to a Teensy 2 that was modified for 3.3v. See here. https://www.pjrc.com/teensy/3volt.html.

### Wiring for ESP8266

```
ESP8266

TX ------------ Teensy RX
RX ------------ Teensy TX
CHPD ---------- 3.3v
```

![Teensy/Feather Wiring](/assets/esp8266_feather/teensy_feather_wiring.jpg)

### Setting up the Teensy

I used the "BareMinimum" sketch from the Arduino IDE on the Teensy. This just contains an empty setup() and loop() method.

## Using PuTTY

Plug in both the Teensy and the feather.

Fire up PuTTY, and connect to the COM port corresponding to the feather at 115200 baud over a serial connection.

When it connects, you should see some gibberish, and a message saying "ready".

![ESP8266 Download Tool Settings](/assets/esp8266_feather/ready_prompt.png)

From here you can enter any of the AT commands. The "Hello World" in this case is the `AT` commands which should just echo "OK"

```
AT

OK
```

Note that for the ESP8266 to register a command, it needs a carriage return (Ctrl + M) and a line feed (Ctrl + J) character.

This means that to enter the AT command, I would type AT, followed by Ctrl + M, and Ctrl + J.

From here, any other commands can be sent to the ESP8266 in this way.

## Using Serial1 on a Teensy using Arduino

Here, I'll be using Teensy with the Arduino IDE. 

The gist of it is that we'll be using Serial1 to communicate to the ESP8266 from the Teensy.

```
void setup() {
    Serial1.begin(115200);
    Serial1.println("AT");
}

void loop() {
}

```

Flash the above code to the Teensy and connect to the ESP8266 through PuTTY. When the code runs on the Teensy, the commands being run should be visible in PuTTY.

# Why?

There we have it! After a few hours of tinkering, I managed to run the AT firmware on this device. 

Do I recommend doing this? Perhaps in a pinch. Clearly, the primary use of this ESP8266 feather would be to have an easily programmable WiFi chip that's compatible with Adafruit's Feather ecosystem. That being said, I could imagine this being useful even within the feather ecosystem, so that some other microcontroller could be given wifi capabilities.
