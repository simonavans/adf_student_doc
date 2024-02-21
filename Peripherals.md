Besides audio processing capabilities, the ESP-32 LyraT board has built-in [peripherals](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/peripherals/esp_peripherals.html) for humans to communicate with the ESP32. Such a peripheral would be the touch keys (Labeled Play, Set, etc.) or a connected SD-card. This section will show how developers might implement this communication in code.

First off, there are many types of peripherals in ESP-ADF. The most relevant ones I could find are:
- Button;
- Touch buttons;
- [SD-card](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/peripherals/periph_sdcard.html);
- WiFi;
- Flash;
- Aux In;
- Console;
- Bluetooth;
- LED;
- Spiffs;
- LCD.
# Peripheral lifecycle
Before using any peripheral, the ESP's peripherals set needs to be initialized. This set manages all peripherals used by the board. Initialization and deinitialization are a very standard procedure, like we've seen before:
```c
// Initialization
esp_periph_config_t periph_cfg = DEFAULT_ESP_PERIPH_SET_CONFIG();
esp_periph_set_handle_t set = esp_periph_set_init(&periph_cfg);

// Deinitialization
esp_periph_stop_all(set);
esp_periph_set_destroy(set);
```
# The [SD-card peripheral](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/peripherals/periph_sdcard.html)
An SD-card can be used to read or write any data for persistent storage in a file format. For most use cases though, developers would want to store music on an SD card to play or record on the board. For this very reason, ESP-ADF introduced playlists. Playlists work almost exactly like they would in the real world; it is a collection of audio files which can be played one after the other. We can combine the functionality of the SD-card API with the playlists API for sound playback on an SD-card. 

This peripheral needs to be initialized as well, just like the peripheral set. After that, a variable can be declared which will be the handle for working with the newly created playlist (this needs to be initialized as well). Then, the playlist can be scanned for audio files using a particular path and file extension filter. The function for this accepts a callback function pointer as parameter, which runs when the scanning is done. The next function will output all found file URLs to audio files to the playlist handle. The playlist is then linked to a FatFs audio element, which can read out the actual data from the SD-card. Of course all of this needs to be cleaned up by the end of the program.
```c
void app_main(void) {
	esp_periph_config_t periph_cfg = DEFAULT_ESP_PERIPH_SET_CONFIG();
	esp_periph_set_handle_t set = esp_periph_set_init(&periph_cfg);

	audio_board_sdcard_init(set, SD_MODE_1_LINE);

	// Create the playlist
	playlist_operator_handle_t sdcard_list_handle = NULL;
	sdcard_list_create(&sdcard_list_handle);

	// Scan the root directory of the SD card with depth 0 (meaning don't search
	// inside nested directories) for files with extension mp3.
	sdcard_scan(on_playlist_scan, "/sdcard", 0, (const char *[]) {"mp3"}, 1, sdcard_list_handle);

	// Send file URLs to the playlist handle
	sdcard_list_show(sdcard_list_handle);

	// Get the URL of the first audio file that plays and store it inside
	// the url variable.
	char *url = NULL;
	sdcard_list_current(sdcard_list_handle, &url);

	fatfs_stream_cfg_t fatfs_cfg = FATFS_STREAM_CFG_DEFAULT();
	fatfs_cfg.type = AUDIO_STREAM_READER;
	fatfs_stream_reader = fatfs_stream_init(&fatfs_cfg);

	// Link the playlist to the FatFs audio element
	audio_element_set_uri(fatfs_stream_reader, url);

	esp_periph_stop_all(set);

	// Clean up playlist
	sdcard_list_destroy(sdcard_list_handle);

	esp_periph_destroy(set);
}
```
## SD-card scanning
Even though most of this procedure seems straightforward, the `sdcard_scan` function might be a little confusing. The function is responsible for scanning the SD-card for audio files, and has a lot of parameters that define how the SD-card is scanned. The function signature is as follows:
`sdcard_scan(void *callback, const char *path, int depth, const char *file_extensions[], int filter_num, void *playlist_handle)`

First is the callback, which is a function we need to define for when a new file URL matching the filters is found. Since you'll probably want to use all of the found files, you should save each file URL to the created playlist. Such a function would look like the following:
```c
void sdcard_url_save_cb(void *user_data, char *url) {
	// Cast void pointer to the correct type
	playlist_operator_handle_t sdcard_handle = (playlist_operator_handle_t)user_data;
	
	esp_err_t ret = sdcard_list_save(sdcard_handle, url);
	if (ret != ESP_OK) {
		ESP_LOGE(TAG, "Fail to save sdcard url to playlist");
	}
}
```

The next parameter is the path, which is `/sdcard` for the root directory. Then the depth, indicating how many levels of nesting directories should be scanned for files. This is followed by the file extensions array, together with `filter_num` (I believe this is always equal to the amount of elements in `file_extensions`). Last is the playlist handle as a void pointer.