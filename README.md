# Introduction to ESP-ADF
A guide that introduces walks through the basic usages of the ESP-ADF framework. It is aimed at students on the Computer Science study at Avans, Netherlands with little to no experience with ESP-ADF and a basic understanding of the C programming language.
# Disclaimer
This guide is not meant to be, nor should it be used as, the single source of truth for ESP-ADF. Always do your own research on the topics explained here. Also, do not assume everything in this guide is correct, as I myself am still learning about ESP-ADF. Anyone is free to use any material from this guide. This guide will be updated for as long as I think it is needed and for as long as the current semester lasts.
# Motivation
This guide was made because of a lack of documentation about ESP-ADF. The [official documentation](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/index.html) does describe the basic building blocks for the framework, but lacks many code examples where they are demonstrated. Most of their documentation consists of individual function documentation, meaning developers need to figure out themselves if and when they should use them. There also seems to be no simple and complete enough guide on BrightSpace (I don't think you're surprised), but [this presentation](https://brightspace.avans.nl/d2l/le/lessons/154524/topics/1206679) seems to be the most useful resource over there. Due to this, I've found myself struggling to understand the framework, and had to go back and forth between documentation and code examples to find out the real purpose of each function.

I expected that many students would have problems understanding ESP-ADF, given that many do not have much prior experience with C, ESP32 frameworks or even embedded software development as a whole. Students need a simpler, better documented and more complete guide to get them to understand the framework better. A link between documentation and code examples is essential to be included in such a guide. With all of this in mind, I decided to make this guide.
# Introduction
Using ESP-ADF, developers can program all kinds of audio processing functionality and deploy them on a variety of ESP32 boards, such as the ESP32 LyraT. Since we are using this board for developing the proftaak product, we will be using this framework to develop audio processing functionality for the smart speaker.

Some use cases could be streaming audio over various protocols, like HTTP, I<sup>2</sup>S (speaker output) and FatFs (SD card file system) to name a few, and playing audio (by connecting an external speaker). Besides input and output of audio, the developer can process ingoing audio in various ways before it becomes output. For example, audio may be resampled, equalized or converted to a different file format. ESP-ADF offers all of these options within just one framework.
# Installation
Before installing ESP-ADF, it is assumed that you have ESP-IDF and some IDE (VS Code, Espressif IDE) installed. There are setup guides available on BrightSpace. Assuming that you're using VS Code as your IDE, you can follow the installation guide below. Here's a link that takes you to the [guide on BrightSpace](https://brightspace.avans.nl/d2l/le/lessons/154524/topics/1225539). Below is a shortened version of that guide:
1. Launch your terminal, then go to your `esp` folder  (usually in `C:\esp` or in `C:\Users\username\esp`) using the `cd` command;
2. Run the following command:
```
git clone --recursive https://github.com/espressif/esp-adf.git
```
This will clone the master branch of the repo and will take some time.
3. In VS Code, press `F1` or `Ctrl` + `Shift` + `p` and find the option `ESP-IDF: Install ESP-ADF`. Then select `Use existing repository`, then `GitHub`. Finally, navigate to the `esp-adf` folder in your `esp` directory.

==If you run into any issues, it is recommended that you follow the linked guide above.==
# Building blocks of ESP-ADF
In basic terms, [ESP-ADF's documentation](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/index.html) describes the foundation of the framework. Though, it only documents the most general components and not much more besides function documentation. However, I still recommend checking it out as it can help you better understand the big picture.
## [Audio element](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/framework/audio_element.html)
In ESP-ADF, each step in the audio processing chain is represented by an audio element. An audio element can be thought of as an entity that takes in some input, processes it, and returns output. Each encoder, decoder, filter, input stream and output stream is represented as an audio element. Audio elements may take in callback functions for various states in its life cycle, such as when the audio element is opened, reading or writing. The type of callbacks an audio element supports depend on the type of audio element.

Frequently used types of stream elements are:
- [FatFs stream](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/streams/index.html#fatfs-stream), for reading from and writing to the file system on an inserted SD card;
- [HTTP stream](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/streams/index.html#http-stream), for reading or writing over the HTTP, HTTPS or HTTP Live Stream protocol;
- [TCP client stream](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/streams/index.html#tcp-client-stream), for reading from and writing to systems on the internet;
- [I<sup>2</sup>S stream](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/streams/index.html#i2s-stream), can communicate with several hardware interfaces on the ESP (like ADC and DAC), but is mostly used for writing to externally connected speakers for playing music;
- [Raw stream](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/streams/index.html#raw-stream), sends and/or receives raw audio samples to/from another stream;
- [SPIFFS stream](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/streams/index.html#spiffs-stream), reads from and writes to the flash memory of the ESP;

Frequently used types of codec elements are:
- [MP3 decoder](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/codecs/mp3_decoder.html), converts MP3 data into raw audio samples.
- [WAV encoder and decoder](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/codecs/wav_codecs.html), converts between WAV data and raw audio samples.

The selection of audio processing elements includes:
- [Downmix](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/audio-processing/downmix.html), combining two audio files or streams to one. The gains of the audio files can be manipulated depending on the type of downmix. The talking clock example project demonstrates downmixing;
- [Equalizer](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/audio-processing/equalizer.html), can set the gain/loudness of individual ranges of frequencies, making it possible to change the volume of specific tones of frequency. This element defines ten frequency ranges;
- [Resample filter](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/audio-processing/filter_resample.html), can change the sample rate per time unit of an audio file (which essentially correlates with the quality of the audio). It can also switch between the amount of channels (2 for stereo, 1 for mono);
- [Sonic](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/audio-processing/audio_sonic.html), can change the pitch and speed of the audio.

> Note: audio elements can be manipulated in ways far beyond what is documented below. However, because these operations are relatively complicated for the proftaak, and because these operations are mostly handled by an audio pipeline, they will not be documented.
### (De)initialization
Before an audio element can be used effectively, it needs to be initialized so it works properly. This is typically done using a single function, like `mp3_decoder_init(...)`. Initialization of audio elements can be done in various ways, as many have tons of options to be set. Knowing each of these options and setting them correctly can be a time consuming task for developers, which is why ESP-ADF offers an easier method using configuration structs.

Upon calling the initialize function of an element, a pointer to a configuration struct can be passed as an argument containing all of the possible configuration options. This way, the developer can be sure that the function receives all of its necessary options. To make things even easier, the framework has built-in default configuration functions that return a configuration struct with many options set to default values. This helps developers so that they do not need to worry about options that are trivial to their program. A code example using a regular configuration struct might look like the following:
```c
mp3_decoder_cfg_t mp3_config = {
	.option1 = "foo",
	.option2 = "bar"
}
// The audio_element_handle_t type symbolizes an audio element as a whole.
audio_element_handle_t mp3_decoder = mp3_decoder_init(&mp3_config);
```

Or, using a default configuration function:
```c
mp3_decoder_cfg_t mp3_config = DEFAULT_MP3_DECODER_CONFIG();
// Optionally, you can override default options of the config later.
mp3_config.task_core = 1;
audio_element_handle_t mp3_decoder = mp3_decoder_init(&mp3_config);
```

Unfortunately, many developers forget a very important step after initialization, which is deinitialization. Since initializing an audio element requires allocating resources (which live for the duration of the program), those resources also need to be deallocated. For each audio element that is initialized, a deinitialization function needs to be called. The `audio_element_deinit(...)` function can deinitialize any audio element. It requires one argument, which is the audio element of type`audio_element_handle_t`. Typically, this function is called after any variables using or referring to audio elements (like audio pipelines and event interfaces) are destroyed or deinitialized. This is to prevent unexpected behaviour.
## [Audio pipeline](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/framework/audio_pipeline.html)
Standalone audio elements are not very useful. It would be better if audio elements could take in the data returned from another audio element, thus creating a chain of audio elements. In ESP-ADF, this is called an audio pipeline. It contains several audio elements able to process data and give their output data to the next audio element in line. One might imagine a scenario where an MP3 music file is read from an SD card and played by external speakers. The speakers cannot directly take in the MP3 file, since the speakers can only work with individual audio samples over the I<sup>2</sup>S protocol. Some required audio elements would be a FatFs element (for reading the SD card), an MP3 decoder and an I<sup>2</sup>S stream (for playing the music). Using an audio pipeline, these elements can be chained to produce the desired output. In pseudocode, that would look like this:
```c
fatfs_element.link_to(&sd_card);
mp3_decoder_element.link_to(&fatfs_element);
i2s_stream_element.link_to(&mp3_decoder_element);
```
In this code, an I<sup>2</sup>S stream element is linked to an MP3 decoder element, which is linked to a FatFs element. This way, when the SD card has new data, it is processed in the chain and will end up at the I<sup>2</sup>S stream. Notice that this linkage is incredibly similar to a linked list data structure.
### Initialization
Just like audio elements, audio pipelines need to be initialized and deinitialized afterwards as well. Initialization for audio pipelines follows an incredibly similar approach compared to audio elements:
```c
audio_pipeline_cfg_t pipeline_cfg = DEFAULT_AUDIO_PIPELINE_CONFIG();
audio_pipeline_handle_t pipeline = audio_pipeline_init(&pipeline_cfg);
```
### Registration and linkage
Next, the audio elements need to be added to the audio pipeline in a specific order. This is done by first registering each audio element on the audio pipeline, and then linking everything together in the pipeline. Beside a pipeline argument and audio element argument, the functions for registering and linking take in a third parameter: `char *name`. This is a string which uniquely identifies an audio element for a given pipeline. This third parameter makes it possible to have multiple audio elements of the same type in different parts of the pipeline.

You can decide in which order the audio elements are placed in the linking part. Here, all of the audio elements' events are subscribed to. The link function takes in an array where index 0 is the first registered audio element, and where the last index is the last registered audio element. The last parameter of the function is an integer that should be equal to the elements in the array.
```c
audio_element_handle_t mp3_decoder = mp3_decoder_init(...);
audio_element_handle_t i2s_stream = i2s_stream_init(...);
audio_pipeline_handle_t pipeline = audio_pipeline_init(...);

audio_pipeline_register(pipeline, mp3_decoder, "mp3");
audio_pipeline_register(pipeline, i2s_stream, "i2s");

// Link elements in order MP3 decoder --> i2s stream
const char *link_tag[2] = {"mp3", "i2s"};
audio_pipeline_link(pipeline, &link_tag[0], 2);
```
### Event listening
```c
// Listens for events from audio elements
audio_pipeline_set_listener(pipeline, your_event_handle);

// Listens for events from peripherals
audio_event_iface_set_listener(your_iface_handle, your_event_handle);
```
### Start, stop, pause and resume
```c
// Starts the audio pipeline
audio_pipeline_run(pipeline);

// Stops the audio pipeline, which takes some time.
// We need to wait before continuing to prevent
// undefined behaviour with the wait function.
audio_pipeline_stop(pipeline);
audio_pipeline_wait_for_stop(pipeline);

// Pauses the audio pipeline
audio_pipeline_pause(pipeline);

// Resumes the audio pipeline
audio_pipeline_resume(pipeline);
```
### Deinitialization
Deinitialization is a little more complex, though. Assuming that, when you want to deinitialize an audio pipeline, the audio pipeline contains linked audio elements, these need to be unlinked first. Also, stopping the audio pipeline when it is running isn't done automatically either. The following code is an example of how an audio pipeline could be cleaned up correctly:
```c
// First, stop the pipeline if it was running
audio_pipeline_stop(pipeline);

// Stopping takes some time. We need to wait before continuing
// to prevent undefined behaviour with this function.
audio_pipeline_wait_for_stop(pipeline);

// Destroy the links to audio elements in the pipeline.
// Note: after this function, the elements are still registered!
audio_pipeline_terminate(pipeline);

// Unregister all audio elements in the pipeline
audio_pipeline_unregister(pipeline, your_element);

// Stop listening to this audio pipeline if you're using
// an event listener.
audio_pipeline_remove_listener(pipeline);

// Deinit pipeline
audio_pipeline_deinit(pipeline);

// Deinit all individual audio elements
audio_element_deinit(element1);
audio_element_deinit(element2);
```

> Note: this is the order of functions in which an audio pipeline was deinitialized in one of the example projects. I do not know if this order of functions works in any case. When in doubt, check the official documentation.
## [Event interfaces/listeners](https://docs.espressif.com/projects/esp-adf/en/latest/api-reference/framework/audio_event_iface.html)
This topic is yet to be documented.
