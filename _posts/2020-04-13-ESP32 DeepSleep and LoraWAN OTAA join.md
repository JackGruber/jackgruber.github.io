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

To store the LMIC configurations we have to define the following variables on a global scope.

```c++
RTC_DATA_ATTR u4_t RTC_LORAWAN_netid = 0;
RTC_DATA_ATTR devaddr_t RTC_LORAWAN_devaddr = 0;
RTC_DATA_ATTR u1_t RTC_LORAWAN_nwkKey[16];
RTC_DATA_ATTR u1_t RTC_LORAWAN_artKey[16];
RTC_DATA_ATTR u1_t RTC_LORAWAN_dn2Dr;
RTC_DATA_ATTR u1_t RTC_LORAWAN_dnConf;
RTC_DATA_ATTR u4_t RTC_LORAWAN_seqnoDn;
RTC_DATA_ATTR u4_t RTC_LORAWAN_seqnoUp = 0;
RTC_DATA_ATTR s1_t RTC_LORAWAN_adrTxPow;
RTC_DATA_ATTR s1_t RTC_LORAWAN_datarate;
RTC_DATA_ATTR u1_t RTC_LORAWAN_txChnl;
RTC_DATA_ATTR s2_t RTC_LORAWAN_adrAckReq;
RTC_DATA_ATTR u1_t RTC_LORAWAN_rx1DrOffset;
RTC_DATA_ATTR u1_t RTC_LORAWAN_rxDelay;
RTC_DATA_ATTR u4_t RTC_LORAWAN_channelFreq[MAX_CHANNELS];
RTC_DATA_ATTR u2_t RTC_LORAWAN_channelDrMap[MAX_CHANNELS];
RTC_DATA_ATTR u4_t RTC_LORAWAN_channelDlFreq[MAX_CHANNELS];
RTC_DATA_ATTR band_t RTC_LORAWAN_bands[MAX_BANDS];
RTC_DATA_ATTR u2_t RTC_LORAWAN_channelMap;
```

To following functions are for storing and loading the variables.

```c++
void LMICSaveVarsToRTC()
{
    Serial.println(F("Save LMIC to RTC ..."));
    RTC_LORAWAN_netid = LMIC.netid;
    RTC_LORAWAN_devaddr = LMIC.devaddr;
    memcpy(RTC_LORAWAN_nwkKey, LMIC.nwkKey, 16);
    memcpy(RTC_LORAWAN_artKey, LMIC.artKey, 16);
    RTC_LORAWAN_dn2Dr = LMIC.dn2Dr;
    RTC_LORAWAN_dnConf = LMIC.dnConf;
    RTC_LORAWAN_seqnoDn = LMIC.seqnoDn;
    RTC_LORAWAN_seqnoUp = LMIC.seqnoUp;
    RTC_LORAWAN_adrTxPow = LMIC.adrTxPow;
    RTC_LORAWAN_datarate = LMIC.datarate;
    RTC_LORAWAN_txChnl = LMIC.txChnl;
    RTC_LORAWAN_adrAckReq = LMIC.adrAckReq;
    RTC_LORAWAN_rx1DrOffset = LMIC.rx1DrOffset;
    RTC_LORAWAN_rxDelay = LMIC.rxDelay;
    memcpy(RTC_LORAWAN_channelFreq, LMIC.channelFreq, MAX_CHANNELS*sizeof(u4_t));
    memcpy(RTC_LORAWAN_channelDrMap, LMIC.channelDrMap, MAX_CHANNELS*sizeof(u2_t));
    memcpy(RTC_LORAWAN_channelDlFreq, LMIC.channelDlFreq, MAX_CHANNELS*sizeof(u4_t));
    memcpy(RTC_LORAWAN_bands, LMIC.bands, MAX_BANDS*sizeof(band_t));
    RTC_LORAWAN_channelMap = LMIC.channelMap;
}

void LMICLoadVarsFromRTC()
{
    Serial.println(F("Load LMIC vars from RTC ..."));
    LMIC_setSession(RTC_LORAWAN_netid, RTC_LORAWAN_devaddr, RTC_LORAWAN_nwkKey, RTC_LORAWAN_artKey);
    LMIC.dn2Dr = RTC_LORAWAN_dn2Dr;
    LMIC.dnConf = RTC_LORAWAN_dnConf;
    LMIC.seqnoDn = RTC_LORAWAN_seqnoDn;
    LMIC_setSeqnoUp(RTC_LORAWAN_seqnoUp);
    LMIC_setDrTxpow(RTC_LORAWAN_datarate, RTC_LORAWAN_adrTxPow);
    LMIC.txChnl = RTC_LORAWAN_txChnl;
    LMIC.adrAckReq = RTC_LORAWAN_adrAckReq;
    LMIC.rx1DrOffset = RTC_LORAWAN_rx1DrOffset;
    LMIC.rxDelay = RTC_LORAWAN_rxDelay;
    memcpy(LMIC.channelFreq, RTC_LORAWAN_channelFreq, MAX_CHANNELS*sizeof(u4_t));
    memcpy(LMIC.channelDrMap, RTC_LORAWAN_channelDrMap, MAX_CHANNELS*sizeof(u2_t));
    memcpy(LMIC.channelDlFreq, RTC_LORAWAN_channelDlFreq, MAX_CHANNELS*sizeof(u4_t));
    memcpy(LMIC.bands, RTC_LORAWAN_bands, MAX_BANDS*sizeof(band_t));
    LMIC.channelMap = RTC_LORAWAN_channelMap;
}
```

After the `LMIC_reset()` we check if we need to load saved data.

```c++
...

os_init();
LMIC_reset();

// Load the LoRa information from RTC
if(RTC_LORAWAN_seqnoUp != 0) { LMICLoadVarsFromRTC(); }

...
```

To check if we can go into DeepSleep, we set a var GOTO_DEEPSLEEP in the event `EV_TXCOMPLETE`
and check in oure loop if we can go to DeepSleep.

```c++
bool GOTO_DEEPSLEEP = false;
void loop()
{
    os_runloop_once();
    int seconds = 300;
    if(!os_queryTimeCriticalJobs(ms2osticksRound( (seconds*1000) - 1000 )))
    {
        Serial.println("Can sleep");
        if(GOTO_DEEPSLEEP == true)
        {
            esp_sleep_enable_timer_wakeup(seconds * 1000000);
            esp_deep_sleep_start();
        }
    }
}
```

{: .box-error}
The duty cycle in LMIC is based on micros() and this is also resedted!
Therefore the dutycycle is not yet covered in this example!

{: .box-note}
[ESP32 TTN environmental sensor](https://github.com/JackGruber/esp32_ttn_environmental_sensor) is a project from me where the code is used.
