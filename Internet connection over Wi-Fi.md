Of course, we would like to access the internet to make the internet radio work on our boards. But before we can use an HTTP stream, we must first connect our ESP to Wi-Fi.

Both ESP-IDF and ESP-ADF have their own APIs to connect to Wi-Fi. Of those two, the ESP-IDF way requires more code to make it work, but allows more control over the Wi-Fi connection than ESP-ADF's Wi-Fi API does. Though the latter option is way less complex, and will be fully demonstrated below. You can look at an [example](https://github.com/espressif/esp-idf/tree/master/examples/wifi/getting_started/station) if you would like to do it the IDF way.
# The ESP-ADF Wi-Fi APi
Before being able to connect to Wi-Fi, two things must be done in code. First is the initialization of the Non-Volatile Storage (NVS). Essentially, this is a key-value based storage mechanism that can store data securely using [NVS encryption](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/storage/nvs_encryption.html). By default, the Wi-Fi API uses NVS to store the network SSID (network name) and password of the network it's connecting to, for an extra layer of security. The second thing that must be done is initializing the TCP/IP stack on the ESP.

Next up is creating and configuring the Wi-Fi peripheral. First, initialize the peripheral set. Then, initialize the Wi-Fi peripheral using a configuration containing the SSID and password of the network you want to connect to. Finally, start the Wi-Fi peripheral and block until it is connected.
```c
// Initialize NVS
esp_err_t err = nvs_flash_init();
if (err == ESP_ERR_NVS_NO_FREE_PAGES) {
	// If NVS partition was truncated and needs to be erased
	ESP_ERROR_CHECK(nvs_flash_erase());

	// Retry
	nvs_flash_init();
}

// Initialize TCP/IP stack
ESP_ERROR_CHECK(esp_netif_init());

// Initialize peripheral set
esp_periph_config_t periph_cfg = DEFAULT_ESP_PERIPH_SET_CONFIG();
esp_periph_set_handle_t set_handle = esp_periph_set_init(&periph_cfg);

periph_wifi_cfg_t wifi_cfg = {
	.wifi_config.sta.ssid = "change_me",
	.wifi_config.sta.password = "change_me"
};
esp_periph_handle_t wifi_handle = periph_wifi_init(&wifi_cfg);

esp_periph_start(set_handle, wifi_handle);
periph_wifi_wait_for_connected(wifi_handle, portMAX_DELAY);
```
It is recommended but not required to put this code before the initialization the board, audio elements, etc.

> Make sure you're not pushing your network SSID and password to your repository for people to see! I recommend you put these values in the SDK config/menuconfig (inside a `Kconfig.projbuild` file that you put in the directory of the component containing the Wi-Fi code). You could then gitignore the `sdkconfig` and `sdkconfig.old` files (or a similar technique that hides these files) contained in the root directory of the project. A `Kconfig.projbuild` file could look like this:
```
menu "Example Configuration"
    
    config WIFI_SSID
        string "Wi-Fi SSID"
        default "change_me"
        help
            SSID (Wi-Fi network name) to connect to.

    config WIFI_PASS
        string "Wi-Fi password"
        default "change_me"
        help
            Wi-Fi password to use for connecting.
    
endmenu
```
In your code, you could then replace the SSID with `CONFIG_WIFI_SSID` and the password with `CONFIG_WIFI_PASS`.