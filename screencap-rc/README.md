###A brief overview of screencap's config options

Lots of the config options just paste into a part of an FFmpeg command. I recommend reading more about certain options in the FFmpeg documentation.

* `executable` - The binary to run, can be absolute path to compiled FFmpeg
* `threads` - Number of threads to use by default
* `cap_fps` - Capture framerate
* `cap_size` - Capture input size
* `cap_offset` - Capture offset (For recording subsections of your screen)
* `cap_device` - Capture device (Xorg screen ID such as `:0.0`)
* `cap_scale` - Dimensions to scale capture to before rendering (Scaling down saves hard drive space)
* `cap_out` - The codec options to use to render the capture. Default is lossless h264 encoding.
* `cap_in` - Replace capture input entirely (Or `( null )` to remove capture input) - this option disables `cap_fps` `cap_size` `cap_device` `cap_scale` and runtime scale options since it's all hardcoded.


* `*_in` - Inputs and their options (Audio, Video)
* `*_out` - Output codecs and their options
* `*_name` - An array of names for each input stream ie: `( audio1 audio2 )`
* `*_map` - Force pick streams to output. Useful for filters and other options that can change the stream name after input. Automatically copied from `*_name` if empty. Use `( null )` to force no maps.


* `video_filter` - Filter video inputs with an FFmpeg filtergraph. This is applied after `cap_scale`, before runtime scale options, and before the final rounding filter for the cap stream. You are free to use `$rounding_filter` to ensure there are no encoding errors with uneven dimension streams you may have created.
* `audio_filter` - Filter audio inputs with an FFmpeg filtergraph.
 
The `screencap-rc` folder contains pre-written config files for various use cases with lots of comments. This should show you a few ways to use these options.