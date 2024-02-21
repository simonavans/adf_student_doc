This modified example is based on the pipeline_sdcard_mp3_control example project by Espressif. It first scans the SD-card (remember to insert it before running) for MP3 audio files. It will play the first audio clip that it finds. When it is done playing, it will automatically clean up and end the program. This is about the simplest ESP-ADF program possible involving the SD-card.

Prerequisites:
- A working SD-card is inserted into the SD-card slot;
- External speakers are connected via the 'PHONE JACK' connector;
- The SD card contains an MP3 audio file in its root directory;
- The audio file's bitrate, bit width and channel amount is supported by the board's audio chip. If the audio stutters, the song's bitrate is probably too high. Bitrate can be changed in audio manipulation programs like Audacity.
```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

#include "audio_element.h"
#include "audio_pipeline.h"
#include "audio_common.h"
#include "fatfs_stream.h"
#include "i2s_stream.h"
#include "mp3_decoder.h"

#include "esp_peripherals.h"
#include "periph_sdcard.h"

#include "board.h"

#include "sdcard_list.h"
#include "sdcard_scan.h"

void sdcard_url_save_cb(void *user_data, char *url) {
    playlist_operator_handle_t sdcard_handle = (playlist_operator_handle_t)user_data;
    sdcard_list_save(sdcard_handle, url);
}

void app_main(void) {
    esp_periph_config_t periph_cfg = DEFAULT_ESP_PERIPH_SET_CONFIG();
    esp_periph_set_handle_t set = esp_periph_set_init(&periph_cfg);

    audio_board_key_init(set);
    audio_board_sdcard_init(set, SD_MODE_1_LINE);

    playlist_operator_handle_t sdcard_list_handle = NULL;
    sdcard_list_create(&sdcard_list_handle);
    sdcard_scan(sdcard_url_save_cb, "/sdcard", 0, (const char *[]) {"mp3"}, 1, sdcard_list_handle);
    sdcard_list_show(sdcard_list_handle);
    char *url = NULL;
    sdcard_list_current(sdcard_list_handle, &url);

    audio_board_handle_t board_handle = audio_board_init();
    audio_hal_ctrl_codec(board_handle->audio_hal, AUDIO_HAL_CODEC_MODE_DECODE, AUDIO_HAL_CTRL_START);

    i2s_stream_cfg_t i2s_cfg = I2S_STREAM_CFG_DEFAULT();
    i2s_cfg.type = AUDIO_STREAM_WRITER;
    audio_element_handle_t i2s_stream_writer = i2s_stream_init(&i2s_cfg);

    mp3_decoder_cfg_t mp3_cfg = DEFAULT_MP3_DECODER_CONFIG();
    audio_element_handle_t mp3_decoder = mp3_decoder_init(&mp3_cfg);

    fatfs_stream_cfg_t fatfs_cfg = FATFS_STREAM_CFG_DEFAULT();
    fatfs_cfg.type = AUDIO_STREAM_READER;
    audio_element_handle_t fatfs_stream_reader = fatfs_stream_init(&fatfs_cfg);
    audio_element_set_uri(fatfs_stream_reader, url);

    audio_pipeline_cfg_t pipeline_cfg = DEFAULT_AUDIO_PIPELINE_CONFIG();
    audio_pipeline_handle_t pipeline = audio_pipeline_init(&pipeline_cfg);
    mem_assert(pipeline);

    audio_pipeline_register(pipeline, fatfs_stream_reader, "file");
    audio_pipeline_register(pipeline, mp3_decoder, "mp3");
    audio_pipeline_register(pipeline, i2s_stream_writer, "i2s");

    const char *link_tag[3] = {"file", "mp3", "i2s"};
    audio_pipeline_link(pipeline, &link_tag[0], 3);

    audio_pipeline_run(pipeline);

    audio_event_iface_cfg_t evt_cfg = AUDIO_EVENT_IFACE_DEFAULT_CFG();
    audio_event_iface_handle_t evt = audio_event_iface_init(&evt_cfg);
    audio_pipeline_set_listener(pipeline, evt);

    while (1) {
        audio_event_iface_msg_t msg;
        audio_event_iface_listen(evt, &msg, portMAX_DELAY);

        audio_element_state_t el_state = audio_element_get_state(i2s_stream_writer);
        if (el_state == AEL_STATE_FINISHED) {
            break;
        }

        if (msg.source_type == AUDIO_ELEMENT_TYPE_ELEMENT && msg.source == (void *)mp3_decoder && msg.cmd == AEL_MSG_CMD_REPORT_MUSIC_INFO) {
            audio_element_info_t audio_info = {0};
            audio_element_getinfo(mp3_decoder, &audio_info);
            i2s_stream_set_clk(i2s_stream_writer, audio_info.sample_rates, audio_info.bits, audio_info.channels);
            continue;
        }
    }

    audio_pipeline_stop(pipeline);
    audio_pipeline_wait_for_stop(pipeline);
    audio_pipeline_terminate(pipeline);

    audio_pipeline_unregister(pipeline, mp3_decoder);
    audio_pipeline_unregister(pipeline, i2s_stream_writer);
    audio_pipeline_unregister(pipeline, fatfs_stream_reader);

    sdcard_list_destroy(sdcard_list_handle);
    audio_pipeline_deinit(pipeline);
    audio_element_deinit(i2s_stream_writer);
    audio_element_deinit(mp3_decoder);
    esp_periph_set_destroy(set);
}
```