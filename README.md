# rtmp-fallback

Experimental piece of crap that outputs an RTMP stream that switches to a fallback video file when the RTMP input fails.  
Basically, it will take an RTMP input and output the RTMP input when possible or the fallback video when it fails.

## Use Cases

- Maintain a Twitch stream alive and not leave viewers with a black screen incase of a technical issue (such as OBS or computer crash or anything of the like)
- Switching of encoder (as long as the decoder can decode it the same) on the fly
- **Useful for multi-streamer marathons**: Anyone can take the relay after someone is done (as long as their output settings are the same)

## How It Works

**This tool doesn't re-encode any video, it is extremely light.** That makes it kinda hard to use and not very modular, and this is why you need to understand how it works before using it.  
RTMP streams are in FLV format. Sadly, FLV files cannot be concatenated at the file level. To work around this, this tool decodes the FLV format **(it doesn't decode the video and audio tracks)** and reassembles them as MPEGTS (.ts) format. This makes concatenating the input stream and fallback videos possible. Then, it reassembles the MPEGTS data as FLV and output it in an RTMP stream.

TL;DR:  
RTMP FLV data (input stream) -> .TS data (concatenable data) -> Input stream OR Fallback .ts file (that is still .TS data) -> RTMP FLV data

**As there is no decoding/encoding of the video and audio tracks, the parameters of the video and audio tracks from the RTMP stream and fallback file must be identical.**  
A way to check for that is using `ffprobe` (part of `ffmpeg`). You should look at the video and audio tracks lines for the rtmp input stream and fallback .ts file and make sure they're (nearly) identical. **The only way to make sure the fallback .ts file and the input RTMP stream are compatible is by testing it anyway, and try to play the output feed in multiple players such as VLC and `ffplay`.**  
Different bitrates and x264 presets should work; different fps may work; different profiles, pixel formats and resolution definitely won't.

## Install

`npm install -g rtmp-fallback`  
```
Usage: rtmp-fallback rtmpInput fallbackFile rtmpOutput
fallbackFile MUST be a .ts file
Options:
        -l: Enable logging in /tmp of rtmpdump/ffmpeg outputs.
        -t ms: Timeout in milliseconds before switching to fallback file. Defaults to 5000ms.
        -d ms: Set fallback video's duration in ms. Will fallback to automatic detection if not set.
```

## WARNING

As I said before, it is an experimental tool that I can only recommend advanced users to use in production. It has only been tested on Linux, with `nginx_rtmp_module` with the `wait_video` and `wait_key` parameters enabled.  
If you're interested in using this tool for your production, throw me a mail or [get in touch on Discord](https://discord.gg/ThePooN).
