> ==TODO test example code==

This is a small example showcasing a controller for a LED strip that supports the RMT protocol. This configuration utilizes a master ESP32-LyraT and a slave ESP-WROOM-32 that communicate with each other using I2C. The slave will then communicate with the LED strip over RMT. This architecture allows easier access to GPIO pins and easier control over the LED strip.

The code for the master ESP32-LyraT:
```c
#include "driver/i2c.h"
#include "esp_err.h"
#include "esp_log.h"

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#define I2C_MASTER_SCL 23
#define I2C_MASTER_SDA 18
#define I2C_SLAVE_ADDR 0x69
#define I2C_FREQ_HZ 100000

enum led_effects { LED_OFF, LED_ON };

static const char* TAG = "LED_MASTER";

void init_strip_master() {
    i2c_config_t i2c_conf_master = {.mode = I2C_MODE_MASTER,
                                    .sda_io_num = 21,
                                    .sda_pullup_en = GPIO_PULLUP_ENABLE,
                                    .scl_io_num = 22,
                                    .scl_pullup_en = GPIO_PULLUP_ENABLE,
                                    .master.clk_speed = 100000};
    ESP_ERROR_CHECK(i2c_param_config(I2C_NUM_0, &i2c_conf_master));
    ESP_ERROR_CHECK(i2c_driver_install(I2C_NUM_0, I2C_MODE_MASTER, 0, 0, 0));
}

void led_strip_command(uint8_t* msg, size_t len) {
    i2c_cmd_handle_t cmd = i2c_cmd_link_create();
    ESP_ERROR_CHECK(i2c_master_start(cmd));

    // First, send the slave address to start communication with
    // the slave
    ESP_ERROR_CHECK(
        i2c_master_write_byte(cmd, I2C_SLAVE_ADDR << 1 | I2C_MASTER_WRITE, 1));

    // Next, send the data to the slave
    ESP_ERROR_CHECK(i2c_master_write(cmd, msg, len, 1));
    ESP_ERROR_CHECK(i2c_master_stop(cmd));
    ESP_ERROR_CHECK(
        i2c_master_cmd_begin(I2C_NUM_0, cmd, 1000 / portTICK_PERIOD_MS));
    i2c_cmd_link_delete(cmd);
}

void app_main() {
    // Send the color white to the first 30 LEDs on the strip
    for (int i = 0; i < 30; i++) {
        // LED effect, LED number, R, G, B
        uint8_t msg[] = {LED_ON, i, 25, 25, 25};
        led_strip_command(msg, sizeof(msg) / sizeof(msg[0]));
        vTaskDelay(100 / portTICK_PERIOD_MS);
    }
}
```

The code for the slave ESP-WROOM-32:
```c
#include "driver/i2c.h"
#include "driver/rmt.h"
#include "esp_err.h"
#include "esp_log.h"

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#include <stdio.h>
#include "led_strip.h"

// GPIO pin for the LED strip
#define RMT_GPIO_NUM 2
#define I2C_SLAVE_SCL 22
#define I2C_SLAVE_SDA 21
#define I2C_SLAVE_ADDR 0x69
#define I2C_FREQ_HZ 100000
#define LED_STRIP_MAX_LEDS 30
#define BUFF_SIZE 128

static const char* TAG = "LED_SLAVE";

enum led_effects { LED_OFF, LED_ON };

static led_strip_t* strip;

void init_strip_slave() {
    // Init I2C
    i2c_config_t i2c_conf_slave = {.mode = I2C_MODE_SLAVE,
                                   .sda_io_num = I2C_SLAVE_SDA,
                                   .sda_pullup_en = GPIO_PULLUP_ENABLE,
                                   .scl_io_num = I2C_SLAVE_SCL,
                                   .scl_pullup_en = GPIO_PULLUP_ENABLE,
                                   .master.clk_speed = I2C_FREQ_HZ,
                                   .slave.addr_10bit_en = 0,
                                   .slave.slave_addr = I2C_SLAVE_ADDR};
    ESP_ERROR_CHECK(i2c_param_config(I2C_NUM_0, &i2c_conf_slave));
    ESP_ERROR_CHECK(i2c_driver_install(I2C_NUM_0, I2C_MODE_SLAVE, 1024, 0, 0));

    // Init RMT
    rmt_config_t rmt_conf = {.rmt_mode = RMT_MODE_TX,
                             .channel = RMT_CHANNEL_0,
                             .gpio_num = RMT_GPIO_NUM,
                             .clk_div = 2,
                             .mem_block_num = 2};
    ESP_ERROR_CHECK(rmt_config(&rmt_conf));
    ESP_ERROR_CHECK(rmt_driver_install(rmt_conf.channel, 0, 0));

    // Init LED strip
    led_strip_config_t strip_conf = LED_STRIP_DEFAULT_CONFIG(
        LED_STRIP_MAX_LEDS, (led_strip_dev_t)RMT_CHANNEL_0);
    strip = led_strip_new_rmt_ws2812(&strip_conf);
    if (!strip) {
        ESP_LOGE(TAG, "failed to create strip");
        return;
    }
    ESP_ERROR_CHECK(strip->clear(strip, 100));
}

void led_on(uint8_t led, uint8_t red, uint8_t green, uint8_t blue) {
    ESP_ERROR_CHECK(strip->set_pixel(strip, led, red, green, blue));
    ESP_ERROR_CHECK(strip->refresh(strip, 100));
}

void leds_clear() {
    ESP_ERROR_CHECK(strip->clear(strip, 100));
}

void app_main(void) {
    init_strip_slave();

    // Will contain the I2C messages between master and slave
    uint8_t buffer[BUFF_SIZE];

    while (1) {
        // First, read i2c buffer containing the LED effect
        // (on or off)
        int len = i2c_slave_read_buffer(I2C_NUM_0, buffer, BUFF_SIZE,
                                        1000 / portTICK_PERIOD_MS);

        if (!len) {
            continue;
        } else if (len < 0) {
            ESP_LOGE(TAG, "i2c_slave_read_buffer failed");
            continue;
        }

        // Switch on the LED effect
        switch (buffer[0]) {
            case LED_OFF:
                leds_clear();
                break;
            case LED_ON:
                // Reads the led number, red, green and blue values
                i2c_slave_read_buffer(I2C_NUM_0, buffer, 4, portMAX_DELAY);
                led_on(buffer[0], buffer[1], buffer[2], buffer[3]);
                break;
        }
    }
}
```