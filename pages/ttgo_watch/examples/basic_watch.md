---
layout: home
title: FreeRTOS Task Notification Mailbox example
---

{% highlight c++ linenos %}
/* adapted from https://github.com/Bodmer/TFT_eSPI/blob/90cabab91ad3ef239ef1c95d97db9ab666153292/examples/160%20x%20128/TFT_Clock_Digital/TFT_Clock_Digital.ino
 * adapted to:
 * - read from RTC
 * - use watch routines to setup backlight and display
 * - center the time regardless of font chosen (1-8)
 */

#define LILYGO_TWATCH_2020_V1

#include <TTGO.h>
#include <WiFi.h>

int16_t hours_font = 8;
int16_t minutes_font = 8;
int16_t seconds_font = 4;

int16_t old_hour = 24;
int16_t old_minute = 61;
int16_t old_second = 61;

int16_t x_middle;
int16_t y_middle;

int16_t hours_half_height;

int16_t hours_x;
int16_t hours_y;

int16_t minutes_x;
int16_t minutes_y;

int16_t seconds_x;
int16_t seconds_y;

TTGOClass * watch;
TFT_eSPI * screen;

void setup(void) {
  watch = TTGOClass::getWatch();
  watch->begin();
  watch->rtc->check();
  watch->bl->adjust(150);

  screen = watch->eTFT;
  screen->fillScreen(TFT_BLACK);
  screen->setTextColor(TFT_WHITE, TFT_BLACK);

  x_middle = screen->width() / 2;
  y_middle = screen->height() / 2;
  hours_half_height = screen->fontHeight(hours_font) / 2;
  hours_x = x_middle - screen->textWidth("00", hours_font);
  hours_y = y_middle - hours_half_height;

  minutes_x = x_middle;
  minutes_y = hours_y;

  seconds_x = x_middle - screen->textWidth("0", seconds_font);
  seconds_y = y_middle + hours_half_height + 10;
}

void loop() {
  int16_t position_x;
  RTC_Date time = watch->rtc->getDateTime();

  watch->eTFT->startWrite();
  // hour
  if (old_hour != time.hour) {
    old_hour = time.hour;
    position_x = hours_x;
    screen->setTextPadding(screen->textWidth("0", hours_font));
    if (time.hour < 10) position_x += screen->drawChar('0', position_x, hours_y, hours_font);
    position_x += screen->drawNumber(time.hour, position_x, hours_y, hours_font);
  }

  // minute
  if (old_minute != time.minute) {
    old_minute = time.minute;
    position_x = minutes_x;
    screen->setTextPadding(screen->textWidth("0", minutes_font));
    if (time.minute < 10) position_x += screen->drawChar('0', position_x, minutes_y, minutes_font);
    position_x += screen->drawNumber(time.minute, position_x, minutes_y, minutes_font);
  }

  // seconds
  if (old_second != time.second) {
    old_second = time.second;
    position_x = seconds_x;
    screen->setTextPadding(screen->textWidth("0", seconds_font));
    if (time.second < 10) position_x += screen->drawChar('0', position_x, seconds_y, seconds_font);
    position_x += screen->drawNumber(time.second, position_x, seconds_y, seconds_font);
  }
  watch->eTFT->endWrite();

  vTaskDelay(250);
}
{% endhighlight %}
