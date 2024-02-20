# Encoding ogg files for iPod

If you use [DownOnSpot](https://github.com/oSumAtrIX/DownOnSpot) to download songs, they are encoded with Vorbis and in an OGG container.

iTunes cannot import these (and the iPod can't play them back), so we need to convert them first.

## Conversion

tl;dr

```
ffmpeg -i song.ogg -map_metadata 0:s:a:0 -acodec aac -b:a 320k -aac_pns 0 -movflags +faststart -vn song.m4a
```

### Explanation

We need to encode them with AAC, and store them in an M4A container, for iTunes / iPod to be able to play them back.

The most vanilla way to do this is:

```
ffmpeg -i song.ogg -c:a aac song.m4a
```

However, this has a couple of problems:

* Won't get the metadata (e.g. song/album/artist title etc.)
* Regular weird high pitch noises when playing back on an iPod

### Metadata

The metadata in the downloaded OGG tracks is stored at the stream level:

```
$ ffprobe -hide_banner Angels.ogg
Input #0, ogg, from 'Angels.ogg':
  Duration: 00:04:25.00, start: 0.000000, bitrate: 328 kb/s
  Stream #0:0: Audio: vorbis, 44100 Hz, stereo, fltp, 320 kb/s
    Metadata:
      title           : Angels
      album           : Life Thru A Lens
      artist          : Robbie Williams
      album_artist    : Robbie Williams
      track           : 4
      disc            : 1
      label           : Universal-Island Records Ltd.
      date            : 1997-01-01
```

However, the AAC encoder in ffmpeg can only store the metadata at the top level (global metadata). `-map_metadata 0:s:a:0` will do this for us, so it is available to iTunes.

### AAC Perceptual Noise Substitution

Most iPods do not support the PNS feature in their AAC decoder, which ffmpeg by default uses. This results in tracks that sound fine on all devices except after syncing to the iPod!

To fix this, we need `-aac_pns 0` .

## References

* https://wiki.archlinux.org/title/IPod
* https://www.reddit.com/r/ipod/comments/r2cti3/squeaking_noises_only_through_the_ipod_and_only/hry0ys5/
