---
layout: post
title: Low power Arduino Pro Mini
subtitle: 
show-img: true
tags: [howto, hack, Arduino, µC, Hardware]
bigimg: "/img/head/arduino.jpg"
---
For various projects where a long runtime with batteries is important I use a modify `Arduino Pro Mini 328 3.3V/8Mhz (ATmega328P)`.
As an example a flashing sketch is used, which turns the LED on for 1 second and off for 8 seconds. 

### Blink sketch
```c++
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  delay(8000);
}
```

To optimize the power consumption the Arduino Pro Mini is put into DeepSleep instead of waiting 8 seconds.

### DeepSleep sketch
```c++
#include "LowPower.h"

void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  LowPower.powerDown(SLEEP_8S, ADC_OFF, BOD_OFF);
}
```

### Modification
In order to optimize the power consumption more, I modify the Arduino Pro Mini.
For this I unsolder the power LED and the regulator. 
The ATmega328P can support voltage levels between 2.7 V and 5.5 V and this is perfect for my projects, because I use 18650 Li-Ion batterys as power source. 
<img src="/img/posts/2019-12-27/arduino_mod.jpg">



### Results overview

Version | State | Consumption @ 3.3V
--- | --- | ---
Unmodified | Active Mode | 6.0 mA
Unmodified | DeepSleep   | 1.5 mA
Modified   | Active Mode | 3.9 mA
Modified   | DeepSleep   | 5.0 µA
  
  
#### Links
* LowPower librarie: [https://github.com/rocketscream/Low-Power](https://github.com/rocketscream/Low-Power)


