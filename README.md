# Screencap
My screencap script allows me to capture video on linux faster than any normal programs can (Such as recordmydesktop, glc etc) by using pure FFmpeg and x11grab.

* It can record and encode 1080p 30fps video realtime with a single CPU core and supports as many threads as you want (default 2)
* It records audio from pulseaudio and allows you to save different audio sources to different audio tracks. This lets you edit microphone commentary separately from system audio.
* Pass `--` as an option and the following options will be sent to the end of the FFmpeg command, for instance in order to stream to twitch.

### Installation
#### Prerequisites
* Pulseaudio sound server
* FFmpeg compiled with:
  * enable-x11grab
  * enable-libpulse
  * enable-libmp3lame
  * enable-libx264
  * enable-filters
  * enable-pthreads  
  There are many more useful options to compile it with (rtmp for streaming, vaapi & vdpau for performance etc) but these are required for the default settings.
* A compositing window manager wouldn't hurt

#### Setup
Place the script somewhere in `$PATH` (I use `~/bin`)

Open the script and scroll down to line 19 (Under "Defaults" comment) and set these values as wished

* **executable**  
  Path to the FFmpeg binary. Replace this with the full path to a custom compiled version if you have one.
* **threads**  
  Number of threads to use to encode the video. Setting this to the maximum can slow games down if they are CPU hogs. Setting this to 1 could result in FFmpeg not being able to keep up on slower machines.
* **fps**  
  Frames per second - self explanatory
* **capture**  
  Video input size as described [in the FFmpeg documentation](http://ffmpeg.org/ffmpeg.html#Video-Options). Exact width/height: `1920x1080` or keyword: `hd1080`
* **scale**  
  Scale filter options as described [in the FFmpeg documentation](http://www.ffmpeg.org/ffmpeg-filters.html#Options). When an option is set to -1 the aspect ratio is maintained (So the default of `w=-1:h=720` scales the video to 720 pixels high while maintaining aspect ratio)
  
  Scaling the video down is not a CPU necessity as a modern gaming system (Or steam machine) can record 1080p @ 30fps on a single core, however the file size at 720p is already 700Mb per minute and I personally like to keep my original footage.
  
  Output size is automatically scaled to even width/height to keep encoders happy.
* **codec_v**  
  The video codec to use
* **codec_v_options**  
  Options to pass to the encoder (Default is ultrafast lossless h.264, works great for screencasting)
* **audioinput**  
  A bash array of options for audio inputs. Allow me to explain my own settings.
  * `-f pulse`
    Sets the audio input format to pulseaudio
  * `-name "name"`
    Sets the audio recorder name. This lets you change input volumes individually in mixers like `pavucontrol`.
  * `-channel_layout "stereo"`
    FFmpeg complains that it has to guess if I leave this out.
  * `-i "mumbo.jumbo"`
    The audio input device as shown in `pactl list short sources`
  You may add other options to this array as well (Such as changing the amount of audio channels per input)
* **codec_a**  
  The audio codec to use
* **map_a**  
  By default FFmpeg will only map 1 video and 1 audio stream to an avi file. By setting this to the number of inputs in `audioinput` you can record from multiple sources to different tracks in the final file. These tracks can then be extracted, or edited in other software.

Run the script with:

    screencap [options] filename

Press `q` to stop recording.

### Syntax
    usage: screencap [options] filename

    OPTIONS:

      -h, --help
        Show this message

      -r, --fps
        Framerate in FPS

      -i, --input
        Input size in WxH or "window" to pick one with xwininfo

      -o, --output
        Output size.-1 is wildcard to maintain aspect ratio eg: `w=-1:h=720`
        or `default` for no scaling

      -f, --filters
        Manual video filters

      --blind
        Disable video recording

      --mute
        Disable audio recording

      -t, --threads
        Number of threads to use

      --
        Stop screencap receiving input and pass all following parameters to command

### Other tips
#### Editors
In my opinion [blender](http://www.blender.org/) with [gimp](http://www.gimp.org/) and [audacity](http://audacity.sourceforge.net/) outperforms all other linux video editing software.

#### Sound not synchronizing
Pulseaudio has a weird tendency to mix up 48khz and 44.1khz, in such a way that FFmpeg flags can't even un-screw it up. To fix this problem, you can edit `/etc/pulse/daemon.conf` and change or uncomment the  `default-sample-rate` and/or `alternate-sample-rate` lines to get the sound back in sync.

Avconv's x11grab device is broken and doesn't support the `-framerate` option - this means that any dropped frames will result in the output being shifted by a small amount of time and eventually desync from the sound. This is why this script no longer supports avconv.


#### Getting FFmpeg
If you are on ubuntu or an ubuntu derivative, you can use a PPA to install FFmpeg.

    apt-add-repository ppa:jon-severinsson/ffmpeg
    apt-get update
    apt-get install ffmpeg

I am not affiliated with or responsible for any FFmpeg PPAs.

#### Compiling FFmpeg
Compiling a custom FFmpeg if your package manager doesn't have a good one is not the hardest thing in the world. That said I'm not going to offer a massive instruction manual here, but instead the basic steps I use to get the git repo, configure it, and build it on my system (debian sid).

    git clone git@github.com:FFmpeg/FFmpeg.git
    cd FFmpeg
    ./configure --arch=amd64 --enable-pthreads --enable-libopencv --enable-librtmp --enable-libopenjpeg --enable-libopus --enable-libschroedinger --enable-libspeex --enable-libtheora --enable-vaapi --enable-runtime-cpudetect --libdir=/usr/lib/x86_64-linux-gnu --enable-libvorbis --enable-zlib --enable-swscale --enable-libcdio --enable-bzlib --enable-libdc1394 --enable-frei0r --enable-gnutls --enable-libgsm --enable-libmp3lame --enable-libpulse --enable-vdpau --enable-libvpx --enable-gpl --enable-x11grab --enable-libx264 --shlibdir=/usr/lib/x86_64-linux-gnu --enable-shared --disable-static --prefix=$HOME/FFmpeg
    make
    make install

The binary should now be found at `~/FFmpeg/bin/FFmpeg`

You may need to run `make install` as root.