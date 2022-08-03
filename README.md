# esp-idf-lora
## What is it
**esp-lora-library** is a C component to be integrated with the ESP-IDF environment for sending and receiving data through a LoRa transceiver based on SX127x IC(s).

The library itself is based on sandeepmistry's **arduino-LoRa** (https://github.com/sandeepmistry/arduino-LoRa) library for Arduino.

## Installation
 - clone this repo into the `components` directory of your project

## Configuration

Pin(s) configuration can be done through ESP-IDF's integrated menuconfig
under `Component config -> LoRa Configuration`. Just type `make menuconfig` to start
interactive configuration

By default, the pins are:

Pin | Signal
--- | ------
CS | IO15
RST | IO32
MISO | IO13
MOSI | IO12
SCK | IO14

## Usage
A simple **sender** program...
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "lora.h"

void task_tx(void *p)
{
   for(;;) {
      vTaskDelay(pdMS_TO_TICKS(5000));
      lora_send_packet((uint8_t*)"Hello", 5);
      printf("packet sent...\n");
   }
}

void app_main()
{
   lora_init();
   lora_set_frequency(915e6);
   lora_enable_crc();
   xTaskCreate(&task_tx, "task_tx", 2048, NULL, 5, NULL);
}

```
Meanwhile in the **receiver** program...
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "lora.h"

uint8_t but[32];

void task_rx(void *p)
{
   int x;
   for(;;) {
      lora_receive();    // put into receive mode
      while(lora_received()) {
         x = lora_receive_packet(buf, sizeof(buf));
         buf[x] = 0;
         printf("Received: %s\n", buf);
         lora_receive();
      }
      vTaskDelay(1);
   }
}

void app_main()
{
   lora_init();
   lora_set_frequency(915e6);
   lora_enable_crc();
   xTaskCreate(&task_rx, "task_rx", 2048, NULL, 5, NULL);
}
```
