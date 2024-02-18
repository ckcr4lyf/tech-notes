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
