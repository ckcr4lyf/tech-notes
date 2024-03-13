# AV1 encoding for BluRays

Assuming you've ffmpeg built with SVT-AV1 already. (TODO: Write a guide for this).

These are my settings. They may not work for you, but generically they seem to get the job done for me.

## Figuring out the crop

This is especially important for movies, since often they have black bars on the top and bottom. Cropping them out will make encoding faster and waste less space.

Note: Some movies have changing aspect ratios! For instance: The Dark Knight (2008). So be careful.

Using ffmpeg, you can get the cropdetect as such:

```
ffmpeg -i Filename.mkv -vf cropdetect,metadata=mode=print -f null -
```

Which will print a lot of messages, looking like:

```
[Parsed_cropdetect_0 @ 0x69ed680] x1:0 x2:1919 y1:138 y2:941 w:1920 h:800 x:0 y:140 pts:76743 t:76.743000 limit:0.094118 crop=1920:800:0:140
[Parsed_metadata_1 @ 0x69f3f80] frame:1840 pts:76743   pts_time:76.743
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.x1=0
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.x2=1919
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.y1=138
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.y2=941
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.w=1920
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.h=800
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.x=0
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.y=140
[Parsed_metadata_1 @ 0x69f3f80] lavfi.cropdetect.limit=0.094118
```

Here, `crop=1920:800:0:140` is what you want.

## Encoding the video

Since I've built `ffav1` as ffmpeg with (and only with) SVT-AV1, I will encode the video independently from audio / subs etc.

Add in the correct cropdetect, or leave it out if there's no cropping.

```
ffav1 -hide_banner -y -i Filename.mkv -vf crop=1920:800:0:140 -map 0:v:0 -c:v:0 libsvtav1 -preset 6 -svtav1-params tune=0:enable-overlays=1:scm=0 -pix_fmt yuv420p10le -g 120 -b:v:0 5000k -pass 1 -an -f null /dev/null && \
ffav1 -hide_banner -y -i Filename.mkv -vf crop=1920:800:0:140 -map 0:v:0 -c:v:0 libsvtav1 -preset 6 -svtav1-params tune=0:enable-overlays=1:scm=0 -pix_fmt yuv420p10le -g 120 -b:v:0 5000k -pass 2 -an Filename_AV1.mkv
```

## Encoding the audio

First, determine the audio formats present on the source media, using something like `mediainfo filename.mkv` , which let's say, gives:

```
Audio #1
ID                                       : 2
Format                                   : MLP FBA 16-ch
Format/Info                              : Meridian Lossless Packing FBA with 16-channel presentation
Commercial name                          : Dolby TrueHD with Dolby Atmos
Codec ID                                 : A_TRUEHD
Duration                                 : 2 h 35 min
Bit rate mode                            : Variable
Bit rate                                 : 3 243 kb/s
Maximum bit rate                         : 5 472 kb/s
Channel(s)                               : 8 channels
Channel layout                           : L R C LFE Ls Rs Lb Rb
Sampling rate                            : 48.0 kHz
Frame rate                               : 1 200.000 FPS (40 SPF)
Compression mode                         : Lossless
Stream size                              : 3.52 GiB (12%)
Title                                    : Dolby TrueHD/Atmos Audio / 7.1 / 48 kHz / 3243 kbps / 24-bit
Language                                 : English
Default                                  : Yes
Forced                                   : No
Number of dynamic objects                : 11
Bed channel count                        : 1 channel
Bed channel configuration                : LFE

Audio #2
ID                                       : 3
Format                                   : AC-3
Format/Info                              : Audio Coding 3
Commercial name                          : Dolby Digital
Format settings                          : Dolby Surround EX
Codec ID                                 : A_AC3
Duration                                 : 2 h 35 min
Bit rate mode                            : Constant
Bit rate                                 : 448 kb/s
Channel(s)                               : 6 channels
Channel layout                           : L R C LFE Ls Rs
Sampling rate                            : 48.0 kHz
Frame rate                               : 31.250 FPS (1536 SPF)
Compression mode                         : Lossy
Stream size                              : 498 MiB (2%)
Title                                    : Compatibility Track / Dolby Digital EX Audio / 5.1-EX / 48 kHz / 448 kbps
Language                                 : English
Service kind                             : Complete Main
Default                                  : No
Forced                                   : No
```

### Extracting an existing audio track

In this example, there is already a decent 5.1ch Doby Digital track (AC3) with a decent bitrate (448kb/s). So we can identify it with mkvmerge and then extract it:

```
$ mkvmerge -i Filename.mkv
File 'Filename.mkv': container: Matroska
Track ID 0: video (AVC/H.264/MPEG-4p10)
Track ID 1: audio (TrueHD Atmos)
Track ID 2: audio (AC-3 Dolby Surround EX)
Track ID 3: subtitles (HDMV PGS)
Track ID 4: subtitles (HDMV PGS)
Track ID 5: subtitles (HDMV PGS)
Track ID 6: subtitles (HDMV PGS)
Track ID 7: subtitles (HDMV PGS)
Track ID 8: subtitles (HDMV PGS)
Track ID 9: subtitles (HDMV PGS)
Track ID 10: subtitles (HDMV PGS)
Track ID 11: subtitles (HDMV PGS)
Track ID 12: subtitles (HDMV PGS)
Track ID 13: subtitles (HDMV PGS)
Chapters: 18 entries
Global tags: 2 entries
```

And extract the track we want with

```
$ mkvextract Filename.mkv tracks 2:Filename_audio.ac3
Extracting track 2 with the CodecID 'A_AC3' to the file 'Filename_audio.ac3'. Container format: Dolby Digital (AC-3)
Progress: 100%
```

### Encoding an existing audio track

If the only audio was some form of raw audio (e.g. Dolby TrueHD / DTS-HD), we can encode it to a compressed format (such as DD5.1) using:

```
ffmpeg -hide_banner Filename.mkv -map 0:a:0 -c:a:0 ac3 -b:a:0 640k Filename_audio.ac3
```

## Extracting the chapters

If the original source had chapters, you can extract them to merge them into the final encode, via:

```
$ mkvextract Filename.mkv chapters Filename_Chapters.xml
```

## Merging the files together

Once you have the video, audio, subtitles, you can merge them together using something like:

```
mkvmerge --title "Movie Name (Year)" \
--track-name "0:" Filename_AV1.mkv \
--track-name "0:English (DD5.1)" --language 0:eng Filename_audio.ac3 \
--track-name "0:English (SDH)" --language 0:eng Subtitles.srt \
-o Movie.Year.1080p.BluRay.DD5.1.AV1-GROUP.mkv
```
