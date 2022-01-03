---
layout: post
title: ESP32 DeepSleep and LoraWAN OTAA join
subtitle:
show-img: true
tags: [ESP32, LoraWAN, Arduino, µC, LMIC, TTN]
cover-img: "/assets/img/head/arduino.jpg"
---

When the ESP32 goes into DeepSleep and wake up, it will not restart the code execution from where it entered DeepSleep,
like an ATmega witch restarts code execution at the same place including all set variables.
The ESP32 starts in the Setup function an reinitializes all variables, this means that all values that have been saved before are lost.  
In combination with LoraWAN, OTAA join and the LMIC library, this results in the ESP32 making a join after each DeepSleep.
This costs unnecessary power in battery operation and the limited LoraWAN Join requests for each device, additionally this is effected the limited airtime.

To prevent this, we have to cache some variables and reload them on reboot.
Storing the data in the EEPROM on the ESP32 is a bad idea because the ESP32 emulates the EEPROM in the flash and the flash memory cells have a limited life span for write operations.
On the ESP32 we use the RTC memory, witch holds the content during DeepSleep.
To use RTC memoy vor a variable we declare the variable with a preefix `RTC_DATA_ATTR` as an example `RTC_DATA_ATTR int bootcount = 0`, the variable bootcount is now preserved during DeepSleep.

To store the LMIC configurations we have to define the following variable on a global scope.

```c++
RTC_DATA_ATTR lmic_t RTC_LMIC;
```

To following functions are for storing and loading the LMIC structure.

```c++
void SaveLMICToRTC(int deepsleep_sec)
{
    RTC_LMIC = LMIC;
    // EU Like Bands

    //System time is resetted after sleep. So we need to calculate the dutycycle with a resetted system time
    unsigned long now = millis();
#if defined(CFG_LMIC_EU_like)
    for(int i = 0; i < MAX_BANDS; i++) {
        ostime_t correctedAvail = RTC_LMIC.bands[i].avail - ((now/1000.0 + deepsleep_sec ) * OSTICKS_PER_SEC);
        if(correctedAvail < 0) {
            correctedAvail = 0;
        }
        RTC_LMIC.bands[i].avail = correctedAvail;
    }

    RTC_LMIC.globalDutyAvail = RTC_LMIC.globalDutyAvail - ((now/1000.0 + deepsleep_sec ) * OSTICKS_PER_SEC);
    if(RTC_LMIC.globalDutyAvail < 0) 
    {
        RTC_LMIC.globalDutyAvail = 0;
    }
#else
    Serial.println("No DutyCycle recalculation function!")
#endif
}

void LoadLMICFromRTC()
{
    LMIC = RTC_LMIC;
}
```

After the `LMIC_reset()` we check if we need to load the saved data.

```c++
...

os_init();
LMIC_reset();

// Load the LoRa information from RTC
if (RTC_LMIC.seqnoUp != 0)
{ 
    LoadLMICFromRTC();
}

...
```

To check if we can go into DeepSleep, we set a var GOTO_DEEPSLEEP in the event `EV_TXCOMPLETE`
and check in oure loop if we can go to DeepSleep.

```c++
void onEvent(ev_t ev)
{
    ...
    case EV_TXCOMPLETE:
        GOTO_DEEPSLEEP = true;
    ...
}
```

```c++
bool GOTO_DEEPSLEEP = false;
void loop()
{
    os_runloop_once();
    int seconds = 300;
    if(!os_queryTimeCriticalJobs(ms2osticksRound( (seconds*1000) )))
    {
        Serial.println("Can sleep");
        if(GOTO_DEEPSLEEP == true)
        {
            SaveLMICToRTC(seconds);
            esp_sleep_enable_timer_wakeup(seconds * 1000000);
            esp_deep_sleep_start();
        }
    }
}
```

A complete example project can be found here [Git project with example code](https://github.com/JackGruber/ESP32-LMIC-DeepSleep-example/).

{: .box-error}
Correcting the dutycycle is only done in EU like band plans! <br/>
Please respect [Fair Access Policy and Maximum Duty Cycle limits](https://www.thethingsnetwork.org/docs/lorawan/duty-cycle.html) !

{: .box-note}

[Git project with example code](https://github.com/JackGruber/ESP32-LMIC-DeepSleep-example/) <br/>
[ESP32 TTN environmental sensor](https://github.com/JackGruber/esp32_ttn_environmental_sensor) is a project from me where the code is used.

#### Updates

* 2020-09-18 Save and load complete LMIC structure and reset DutyCyle
* 2020-11-26 Correct DutyCyle calculation for LMIC EU like band planes
* 2022-01-03 Reload condition for LMIC corrected
