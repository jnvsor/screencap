# Screencap
* It can record and encode 1080p 30fps video realtime with a single CPU core and supports as many threads as you want (default 2)
* It records audio from pulseaudio and allows you to save different audio sources to different audio tracks. This lets you edit microphone commentary separately from system audio.
* With multiple config files you can have drastically different recording settings for different purposes.
* Complex filters allow anything from rescaling to overlaying webcam in realtime.
* Pass `--` as an option and the following options will be sent to the end of the FFmpeg command, for instance in order to stream to twitch.

### Installation
#### Prerequisites
* FFmpeg  
  **Note:** FFmpeg must be compiled with certain options (Support for x11grab, libmp3lame, libx264, filters, pthreads, pulseaudio, potentially more depending on your config) so if yours doesn't work, [try compiling one yourself](#compiling-ffmpeg).
* Pulseaudio sound server
* A compositing window manager wouldn't hurt

#### Setup
1. Place the script somewhere in `$PATH` (I use `~/bin`)
2. Place the `screencap-rc` folder wherever you want it (I use `~/.sc-rc`)
3. Open the script, scroll down to line 17 and set `config_folder` to the location of your `screencap-rc` folder (In my case that would be `config_folder="$HOME/.sr-rc"`. Don't leave a trailing slash.
4. Run the script with: `screencap [preset] [options] filename`
5. Press `q` to stop recording.
6. Create your own config files. The files in `screencap-rc` are heavily commented to show a few use cases.

### Syntax
    usage: screencap [preset] [options] filename

    PRESET:
      Name of a file in the folder:
        screencap-rc
      or absolute path to equivalent file. Default is:
        screencap-rc/default

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

#### <a name="compiling-ffmpeg"></a>Compiling FFmpeg
Compiling a custom FFmpeg if your package manager doesn't have a good one is not the hardest thing in the world. That said I'm not going to offer a massive instruction manual here, but instead the basic steps I use to get the git repo, configure it, and build it on my system (debian sid).

    git clone https://github.com/FFmpeg/FFmpeg.git
    cd FFmpeg
    ./configure --arch=amd64 --enable-pthreads --enable-libopencv --enable-librtmp --enable-libopenjpeg --enable-libopus --enable-libschroedinger --enable-libspeex --enable-libtheora --enable-vaapi --enable-runtime-cpudetect --enable-libvorbis --enable-zlib --enable-swscale --enable-libcdio --enable-bzlib --enable-libdc1394 --enable-frei0r --enable-gnutls --enable-libgsm --enable-libmp3lame --enable-libpulse --enable-vdpau --enable-libvpx --enable-gpl --enable-x11grab --enable-libx264 --enable-filters
    make -j4

The binary can now be found at `FFmpeg/ffmpeg`
