---
layout: home
title: FreeRTOS Task Notification Mailbox example
---

{% highlight c++ linenos %}
#define millis_print(string) Serial.printf("%lu - " string "\n", millis());

enum {
  task_one = 0x1,
  task_two = 0x2,
  task_three = 0x4
};

TaskHandle_t consumer;

void setup() {
  Serial.begin(115200);

  // create the consumer
  xTaskCreate(
    [](void* _) {
      uint32_t notification_value;

      for (;;) {
        auto received = xTaskNotifyWait(0xffffffff, 0xffffffff, &notification_value, 10000);
        if (received != pdTRUE) {
          millis_print("no notification received");
          continue;
        }

        switch(notification_value) {
          case task_one: millis_print("task one received"); break;
          case task_two: millis_print("task two received"); break;
          case task_three: millis_print("task three received"); break;
          default: millis_print("unknown notification");
        }

        vTaskDelay(20 / portTICK_PERIOD_MS);
      }
    },
    "consumer", 10000, NULL, 30, &consumer
  );

  // create three producers
  xTaskCreate(
    [](void* _) {
      for (;;) {
        millis_print("task one sending");
        xTaskNotify(consumer, task_one, eSetValueWithOverwrite);
        vTaskDelay(random(100) / portTICK_PERIOD_MS);
      }
    },
    "consumer", 10000, NULL, 30, NULL
  );

  xTaskCreate(
    [](void* _) {
      for (;;) {
        millis_print("task two sending");
        xTaskNotify(consumer, task_two, eSetValueWithOverwrite);
        vTaskDelay(random(500) / portTICK_PERIOD_MS);
      }
    },
    "consumer", 10000, NULL, 30, NULL
  );

  xTaskCreate(
    [](void* _) {
      for (;;) {
        millis_print("task three sending");
        xTaskNotify(consumer, task_three, eSetValueWithOverwrite);
        vTaskDelay(random(1000) / portTICK_PERIOD_MS);
      }
    },
    "consumer", 10000, NULL, 30, NULL
  );
}

void loop() {
  vTaskDelay(portMAX_DELAY);
}

{% endhighlight %}
