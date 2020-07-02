---
layout: home
title: TTGO 2020 V1 Notes
---

[Schematic Diagram](assets/t_watch-2020v1-schematic.pdf)
&middot;
[Library Source Code](https://github.com/Xinyuan-LilyGO/TTGO_TWatch_Library)
&middot;
[ESP Arduino](https://github.com/espressif/arduino-esp32)
&middot;
[My Watch Source](https://github.com/red-fox-star/ttgo_watch)

## Power Conservation
- Out of the box the watch will consume ~140mA running most of the example code

### Backlight / Display power off
- Disabling the backlight with `watch->closeBL()` will save almost 90mA
- Sleeping the display with `watch->displaySleep()` saves another 8-10mA

I'm able to get something close to 50mA standby current with the display off and processor at full speed reading the touch interface -- touch to wake works well in this configuration.

### Processor Configuration

- 40mHz crystal clock, validated with `getXtalFrequencyMhz()`
- Valid processor frequencies: 240, 160, 80, 40, 20, 10
- Switching processor frequencies frequently messes with everything and will stall the watch.

Set clock speed with `setCpuFrequencyMhz(160)`. Downscaling the CPU helps a bit with power usage but not as much as I'd expected. With the [basic pong game][1] running, I observe these numbers. There's enough noise in the numbers that the moving window average is still chaotic with a window size of 50 samples. I measured these while the watch was plugged into USB power.

| Frequency | min mA | avg mA | max mA |
|-----------|--------|--------|--------|
| 10mHz     |  43mA  | 110mA  | 245mA  |
| 20mHz     |  35mA  | 115mA  | 247mA  |
| 40mHz     |  34mA  | 115mA  | 243mA  |
| 80mHz     |  36mA  | 130mA  | 252mA  |
| 160mHz    |  34mA  | 114mA  | 249mA  |
| 240mHz    |  21mA  | 145mA  | 251mA  |

Ideally I'd like to be able to downscale the CPU to 10mHz for low power, and then scale back up to 240mHz for a responsive user interface. After a time or two scaling down and back up, the watch becomes unresponsive without throwing a stack trace. I suspect this is because the Serial, I2C, and/or other configuration is relative to the CPU frequency and needs to be reinitialized when the clock multiplier changes. Measuring this out of circuit would be considerably more helpful.

Anecdotally the graphics performance between 160mHz and 240mHz is minimal and even 80mHz produces a barely noticable slowdown. For most circumstances I suspect 80mHz would be sufficient.

### Monitoring Power Consumption

I'm refusing to tear down my watch to monitor the current coming out of the battery. Sacrificing some amount of accuracy I'm using the onboard ADC of the AXP202 to measure it. In order to get some idea about the sleep-time consumption I setup a [moving average](https://github.com/red-fox-star/ttgo_watch/blob/84ba28be1b5ec415e124385ae82782faa7d19ecb/src/moving_average.h) to track it.

{% highlight c++ linenos %}
// enable the axp202 to watch the power lines
watch->power->adc1Enable(
    AXP202_VBUS_VOL_ADC1
    | AXP202_VBUS_CUR_ADC1
    | AXP202_BATT_CUR_ADC1
    | AXP202_BATT_VOL_ADC1
, true);

// every so often, check the levels
battery_current.insert(watch->power->getBattDischargeCurrent());
vbus_current.insert(watch->power->getVbusCurrent());


// show usage on the screen
char display[255];

snprintf(display, sizeof(display),
    "%i%% %3.1fV %4.1fmA / %3.1fV %4.1fmA",
    watch->power->getBattPercentage(),
    watch->power->getBattVoltage() / 1000,
    battery_current.average(),
    watch->power->getVbusVoltage() / 1000,
    vbus_current.average()
);

screen->fillRect(0, 0, 240, 20, TFT_BLACK);
watch->eTFT->drawString(display, 0, 0, 2);
{% endhighlight %}

[1] https://github.com/red-fox-star/ttgo_watch/blob/84ba28be1b5ec415e124385ae82782faa7d19ecb/src/pong.cpp
