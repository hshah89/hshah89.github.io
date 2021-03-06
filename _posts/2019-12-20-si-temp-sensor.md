---
layout: post
title: "Si7021: Sensor"
description: "A how to guide on interfacing the Si7021 temperature/humidity sensor to an Arduino"
date: 2019-12-20
tags: [Arduino, sensor]
image: "02_si7021_cover.jpg"
comments: false
share: false
---
### Temperature and Humidity Sensor

The Si-7021 is a temperature and humidity sensor from Si-Labs. It comes in a 3mm x 3mm DFN package. A breakout board with the necessary pullup resistors and voltage translator ICs can be purchased online at various places. I bought mine from AliExpress. 

## Parts needed

| Part number         | Quantity  | Where to Buy |
| ------------- |---------------| ----------------|
| BME280         | 1            |  [Link](https://www.aliexpress.com/item/4000049691079.html?spm=a2g0o.productlist.0.0.164f2d11Ezrp2n&algo_pvid=96c51089-acf9-4f37-b501-eb99ac0ee869&algo_expid=96c51089-acf9-4f37-b501-eb99ac0ee869-1&btsid=1647e6e2-1717-4021-8e39-fc2b90cad320&ws_ab_test=searchweb0_0,searchweb201602_9,searchweb201603_55)|
| Arduino Nano            | 1             | |


The sensor is communicated via I2C. Below is an image of what the board looks like. The address for the device is already set internally and cannot be changed.

<img src="{{ "/images/si7021_arduino_driver/si7021_hookup.jpg" | prepend: site.baseurl }}" align="middle">

## Parts needed

| Part number         | Quantity  |
| ------------- |---------------|
| Si7021         | 1            |
| Arduino Nano            | 1             |

***

The table also shows the connection to an Arduino.

<table>
						<thead>
							<tr>
								<th>Pin</th>
								<th>Name</th>
								<th>I/O Type</th>
								<th>Description</th>
								<th>I2C</th>
								<th>Arduino Nano</th>
							</tr>
						</thead>
						<tbody>
							<tr>
								<td>1</td>
								<td>VCC</td>
								<td>In</td>
								<td>Voltage in pin: <b>1.7V to 3.6V</b></td>
								<td>VCC</td>
								<td><b>+3.3V</b></td>
							</tr>
							<tr>
								<td>2</td>
								<td>GND</td>
								<td>In</td>
								<td>Ground pin</td>
								<td>GND</td>
								<td><b>GND</b></td>
							</tr>
							<tr>
								<td>3</td>
								<td>SCL</td>
								<td>In</td>
								<td>Serial Clock</td>
								<td>SCL</td>
								<td><b>A5</b></td>
							</tr>
							<tr>
								<td>4</td>
								<td>SDA</td>
								<td>In/Out</td>
								<td>Serial Data In/Out</td>
								<td>SDA</td>
								<td><b>A4</b></td>
							</tr>
						</tbody>
					</table>

Link to my Si7021 drivers for the Arduino can be found on my GitHub page: [Here](https://github.com/hshah89/si7021)

The first part is to include the Si7021 library.
``` cpp
#include <Wire.h>
#include "si7021.h"
```
The next part is to define the baud-rate for the serial port. The device address is already set internally.
``` cpp
#define BAUD_RATE 115200
```
After setting up the library, baud-rate, and the device address, declare the global variables that will be used throughout the program. At a minimum, you need to declare the object Si7021 to use the sensor library.
``` cpp
float temperature; // Storing measured temperature reading
float humidity;    // Storing measured humidity reading

uint8_t chip_id; // Storing SI7021 Chip ID

SI7021 sensor_si7021;        
```
Next is to write the setup function. Here, the I2C bus and the Serial port are initialized. The Si7021 sensor is started up as well. We look for the Chip ID to make sure the device is connected. Once the device is detected, we check for the firmware version for the board. This is optional. I only put that in there to make sure the read/write functionality of the device is working.

``` cpp
void setup()
{

    Wire.begin();
    Serial.begin(BAUD_RATE);

    Serial.println("SI-7021 - Temperature and Humidity Sensor\n");
    chip_id = sensor_si7021.begin();
    if (chip_id)
    {
        Serial.println("Deviced Detected:\tSi7021");
    }
    Serial.print("Firmware Version:\t");
    if (sensor_si7021.get_fw_rev() == 0xFF)
    {
        Serial.print("1.0");
    }
    else if (sensor_si7021.get_fw_rev() == 0x20)
    {
        Serial.print("2.0");
    }

    Serial.println("\n\nTemp\t\tHumidity");
    Serial.println("----------------------------------------------\r");
    getData();
}    
```
The **`getData()`** Arduino function is shown below. It gets the temperature and humidity. This function is declared so you don't have to keep repeating same commands over and over.
``` cpp
void getData()
{
    humidity = sensor_si7021.read_rel_humid();
    temperature = sensor_si7021.read_temp();
    Serial.print(temperature);
    Serial.print(" ºF \t");
    Serial.print(humidity);
    Serial.print(" % \t");
    Serial.print("\n");
}
```
Finally, the loop function is configured. You call the **`getData()`** function and then delay it a certain amount.
``` cpp
void loop()
{
  delay(10e3);
  getData();
}

```
If everything runs correctly, you should get the following output after programming the Arduino board:
``` txt
Deviced Detected:	Si7021
Firmware Version:	2.0

Temp		Humidity
----------------------------------------------
75.43 ºF 	18.25 % 	
75.45 ºF 	17.82 % 	
```
That should get you going with the sensor. Now that it's working with the Arduino board, you can use this sensor with any project. Stay tuned for a projected using this sensor.

Thanks for reading!