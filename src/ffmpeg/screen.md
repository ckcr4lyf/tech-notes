# Encoding Instant Replay to share online

Nvidia has a feature called instant replay that saves `x` minutes of gameplay. This is saved at a fairly high bitrate. If you want to share a part of it with a friend online, then the file is way too big. Additionally, when sharing online, it is possible to get the context at a much lower resolution (e.g. 720p vs 1440p). Also when gaming on an ultrawide, if only the 16:9 part is relevant, then a crop also helps.

## General formula

Ignoring scaling down to 720p and cropping, there are two main codecs that should be used:

* H.264 + AAC, good combo for sharing on Whatsapp etc.
* VP9 + Opus, good combo for sharing on Discord etc.

VP9 is more efficient and will give better results compared to H.264, but slightly less compatibility.

### H.264

The general recipe I like to use is:

```
ffmpeg -i source.mp4 -c:v libx264 -b:v 2000k -c:a aac -b:a 96k clip.mp4
```

### VP9

The general recipe I like to use is:

```
ffmpeg -i source.mp4 -c:v libvpx-vp9 -b:v 2000k -c:a libopus -b:a 96k clip.webm
```

## Additional filters

To crop from my 3440x1440 ultrawide down to 16:9 (useful when the overlay is ultrawide but actual content is 16:9), I use: `-vf crop=2560:1440:440:0`

To resize down to 720p, I use: `vf scale=1280:-1`

Usually I will combine the two, and encode at 1500kbps. If you need to seek to a particular start point, use the `-ss` flag. And if you only want the next `n` seconds, use the `-t` flag. This example starts from the 4 minute mark and takes the next 10 seconds:

```
ffmpeg -ss 00:04:00 -i "source.mp4"  -c:v libvpx-vp9 -b:v 1500k -vf "crop=2560:1440:440:0,scale=1280:-1" -preset veryslow -c:a libopus -b:a 96k -t 10 output.webm
```
