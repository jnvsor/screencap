# Screencap
My screencap script allows me to capture video on linux faster than any normal programs can (Such as recordmydesktop, glc etc) by using pure avconv (ffmpeg) and x11grab.

* It can record and encode 1080p 30fps video realtime with a single CPU core and supports as many threads as avconv does (default 2)
* It records audio from pulseaudio and allows you to save different audio sources to different audio tracks. You can then split off these audio tracks and effectively edit commentary after recording.
* Pass `--` as an option and the following options will be sent to the end of the avconv command, for instance, in order to stream to twitch.

### Installation
#### Prerequisites
* Pulseaudio sound server
* avconv or ffmpeg compiled with:
  * enable-x11grab
  * enable-libpulse
  * enable-libmp3lame
  * enable-libx264
  * enable-filters
  * enable-pthreads  
  There are many more useful options to compile it with (rtmp for streaming, vaapi & vdpau for performance etc) but these are required for the default settings. Check if your distribution's package has these options with `avconv -loglevel 99`
* A compositing window manager wouldn't hurt

#### Setup
Place the script somewhere in `$PATH` (I use `~/bin`)

Open the script and scroll down to line 19 (Under "Defaults" comment) and set these values as wished

* **executable**  
  Path to the avconv/ffmpeg binary. Replace this with the full path to a custom compiled version if you have one.
* **fps**  
  Frames per second - self explanatory
* **capture**  
  Video input size as described [in the avconv documentation](http://libav.org/avconv.html#Video-Options). Exact width/height: `1920x1080` or keyword: `hd1080`
* **scale**  
  Scale filter options as descriped [in the avconv documentation](http://libav.org/avconv.html#scale-1). When an option is set to -1 the aspect ratio is maintained (So the default of `w=-1:h=720` scales the video to 720 pixels high while maintaining aspect ratio)
  
  Scaling the video down is not a CPU neccesity as a modern gaming system (Or steam machine) can record 1080p @ 30fps on a single core, however the file size at 720p is already 700Mb per minute and I personally like to keep my original footage.
  
  Output size is automatically scaled to even width/height to keep encoders happy.
* **preset**  
  Which libx264 recording preset to use. These should be delivered with avconv and `lossless_ultrafast` has never steered me wrong. (Those of you with rediculous machines may want to up the thread count and lower the encoding speed to produce smaller file sizes at the cost of CPU time)
* **codec_v**  
  The video codec to use (Note that the presets delivered with avconv only work with libx264, so you may have to write your own and store it in `~/.avconv` if you plan on using a different encoder)
* **audioinput**  
  A bash array of options for audio inputs. Allow me to explain my own settings.
  * `-f pulse`
    Sets the audio input format to pulseaudio
  * `-name "name"`
    Sets the audio recorder name. This lets you change input volumes individually in mixers like `pavucontrol`.
  * `-i "mumbo.jumbo"`
    The audio input device as shown in `pactl list short sources`
  You may add other options to this array as well (Such as changing the amount of audio channels per input)
* **codec_a**  
  The audio codec to use
* **map_a**  
  By default avconv will only map 1 video and 1 audio stream to an avi file. By setting this to the number of inputs in `audioinput` you can record from 2 different sources to 2 different tracks in the final file. These tracks can then be extracted for, or editted in other software.
* **threads**  
  Number of threads to use to encode the video. Setting this to the maximum can slow games down if they are CPU hogs. Setting this to 1 could result in avconv not being able to keep up on slower machines.

Run the script with:

    screencap [options] filename

Use sigint (Ctrl+C) to stop recording.

### Syntax
    usage: `basename $0` [options] filename

    OPTIONS:

      -h, --help
        Show this message

      -r, --fps
        Framerate in FPS

      -i, --input
        Input size in WxH or "window" to pick one with xwininfo

      -o, --output
        Output size in W:H (-1 wildcard to maintain aspect
        ratio eg: -1:720) or default for no scaling

      -p, --preset
        avconv video preset to use

      -f, --filters
        Manual avconv video filters

      --blind
        Disable video recording

      --mute
        Disable audio recording

      -t, --threads
        Number of threads to use

      --
        Stop screencap recieving input and pass all following flags to avconv

### Other tips
#### Editors
Kdenlive development has ceased. If blender wasn't the best linux video editor before, it sure is now. In my opinion [blender](http://www.blender.org/) with [gimp](http://www.gimp.org/) and [audacity](http://audacity.sourceforge.net/) outperforms all other linux video editing software.

#### Sound not synchronizing
Pulseaudio has a wierd tendancy to mix up 48khz and 44.1khz, in such a way that avconv/ffmpeg flags can't even un-screw it up. To fix this problem, you can edit `/etc/pulse/daemon.conf` and change or uncomment the  `default-sample-rate` and/or `alternate-sample-rate` lines to get the sound back in sync.

#### Compiling avconv
Compiling a custom avconv if your package manager doesn't have a good one is not the hardest thing in the world. That said I'm not going to offer a massive instruction manual here, but instead the basic steps I use to get the git repo, configure it, and build it on my system.

    git clone git://git.libav.org/libav.git
    cd libav
    ./configure --arch=amd64 --enable-pthreads --enable-librtmp --enable-libopenjpeg --enable-libopus --enable-libschroedinger --enable-libspeex --enable-libtheora --enable-vaapi --enable-runtime-cpudetect --libdir=/usr/lib/x86_64-linux-gnu --enable-libvorbis --enable-zlib --enable-swscale --enable-libcdio --enable-bzlib --enable-libdc1394 --enable-frei0r --enable-gnutls --enable-libgsm --enable-libmp3lame --enable-libpulse --enable-vdpau --enable-libvpx --enable-gpl --enable-x11grab --enable-libx264 --shlibdir=/usr/lib/x86_64-linux-gnu --enable-shared --disable-static --prefix=$HOME/libavbuild
    make
    make install