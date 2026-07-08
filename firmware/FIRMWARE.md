# Firmware
> ## Waiting to complete firmware code until I have the required hardware so this is completely untested, but here's a complete rundown of the code's logic:

## Module level:
The module has 4 input pins (INPUT1-4) relevant for controlling the motors. It also has 1 Hall effect sensor output pin, and a 5V and GND pin connected to the power supply (irrelevant to code).

<img src="../imgs/modpcb.png">

Modules 1-3 is controlled and connected to the ESP32
Modules 4-6 is controlled by Arduino Uno 1, 7-9 is controlled by Uno 2.
Modules 10-18 are controlled by the Arduino Mega.

Where the 5 pins are connected to and where each module should be soldered to are labeled on the Main PCB's silkscreen (MOD*), (HALL is the hall effect sensor's input, IN1-4 is INPUT1-4).

<img src="../imgs/mainpcb.png">

## Retrieving data
The ESP32 sends RESTful GET requests to [AviationStack](https://aviationstack.com/)'s [/v1/timetable](https://docs.apilayer.com/aviationstack/docs/aviationstack-api-v-1-0-0#/default/getFlightSchedule) endpoint for the saved flight code. (Note: Free version is limited to 100 req/mo)
This uses the HTTPClient library (dependencies: Wifi.h, HTTPClient.h, WifiClientSecure.h, ArduinoJson.h)

## Parsing
As shown in README.md, there's 18 characters.
Char 1 (Logo): Have a set array of up to 44 of your most likely to be used airlines' IATA code (e.g. "CX"). From response["data"]["airline"]["iataCode"], search for that IATA code from the array, if found, return the index, else return 0 (blank).
Char 2 (IATA): response["data"]["airline"]["iataCode"][0]
Char 3: response["data"]["airline"]["iataCode"][1]
Char 4 (flight code): response["data"]["flight"]["number"][0]
Char 5-7: response["data"][... ...][1-3]
Char 8-10: response["data"]["departure"]["iataCode"][0-2]
Char 11-13: response...["arrival"][...][0-2]
Char 14-18: 
In case of showing time: response...["departure"]["scheduledTime"].substring(11,16) or response...["arrival"]... in if flight is already in air.
In case of showing flight status: Have 2 arrays of different statuses and the shortened version (e.g. ["boarding", "delayed", ...] ["BRDNG", "DELAY", ...]. Get response...["status"], get index of it in the first array, get the value of that index at the 2nd array.

Parse the value of that string substringed [0-5]


## Displaying
For the modules directly connected to the ESP32, the ESP32 uses AccelStepper to control the motors' position, movement, and acceleration. AccelStepper allows for simultanious commands while the default Stepper lib only allows for async execution.
The motor shall rotate until the hall effect sensor sends a signal (using INPUT_PULLUP), which then sets the index to 0. There shall be an array of characters (e.g. [BLANK/DEFAULT, A, B, C, ..., 8, 9, 0, -, :, " ", ...]) with 45 items, program starts a for loop until it reaches the desired character, for each loop, the motor rotates by 8º (AccelStepper allows for motor control by degrees), and increases the counter by 1.

For modules that aren't connected directly to the ESP32, the ESP32 shall send the desired character to be displayed to the Arduino Unos or Mega via I²C, and then it does the above.

## Input
There's a 
