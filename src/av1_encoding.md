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

