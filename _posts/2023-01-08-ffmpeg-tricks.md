---
layout: post
title: 'FFmpeg Tricks'
subtitle: ''
author: 'Will'
header-style: text
lang: en
tags:
  - FFmpeg
  - Tricks
  - 笔记
---

## Scenarios

- Generate gif

```bash
ffmpeg -i <input_file> -ss <start_time> -t <length> -vf "fps=15,scale=-1:360:flags=spline,split[a][b];[a]palettegen[p];[b][p]paletteuse" <output_file>
```

- Generate thumbnail

> Generic

```
+-----+-----+-----+-----+
|  1  |  2  |  3  |  4  |
+-----+-----+-----+-----+
|  5  |  6  |  7  |  8  |
+-----+-----+-----+-----+
|  9  |  10 |  11 |  12 |
+-----+-----+-----+-----+
|  13 |  14 |  15 |  16 |
+-----+-----+-----+-----+
```

`get_thumbnail.sh`

```bash
#!/bin/bash
if [[ -z $1 || ! -f $1 ]]; then
echo "Please provide a valid file name!"
exit 1
fi
[[ -d tmp ]] || mkdir tmp

step=$(echo $(ffprobe -v quiet -show_entries format=duration -print_format csv=p=0 $1) / 17 | bc -lq)
for i in {1..16}; do
pos=$(echo $step '*' $i | bc -lq)
ffmpeg -v fatal -ss $pos -i $1 -frames:v 1 "tmp/$i.png"
done
ffmpeg -v fatal -i "tmp/%d.png" -filter_complex "tile=4x4,scale=1920x1080" $1.jpg

if [[ -f "$1.jpg" ]]; then
echo "$1.jpg generated!"
rm -rf tmp
else
echo "Some error occurred!"
exit 1
fi
```

```bash
./get_thumbnail.sh <input_file>
```

> Using imagemagick

```
+-----+-----+-----+-----+
|  1  |  2  |  3  |  4  |
+-----+-----+-----+-----+
|  5  |  6  |           |
+-----+-----+     7     +
|           |           |
+     8     +-----+-----+
|           |  9  |  10 |
+-----+-----+-----+-----+
|  11 |  12 |  13 |  14 |
+-----+-----+-----+-----+
```

`get_thumbnail_imagemagick.sh`

```bash
#!/bin/bash
if [[ -z $1 || ! -f $1 ]]; then
echo "Please provide a valid file name!"
exit 1
fi
[[ -d tmp ]] || mkdir tmp

step=$(echo $(ffprobe -v quiet -show_entries format=duration -print_format csv=p=0 $1) / 15 | bc -lq)
for i in {1..14}; do
pos=$(echo $step '*' $i | bc -lq)
ffmpeg -v fatal -ss $pos -i $1 -frames:v 1 "tmp/$i.png"
done
cd tmp
magick \( 1.png 2.png 3.png 4.png +append -resize 50% \) \
\( \
\( \( 5.png 6.png +append -resize 50% \) 8.png -append \) \
\( 7.png \( 9.png 10.png +append -resize 50% \) -append \) \
+append \
\) \
\( 11.png 12.png 13.png 14.png +append -resize 50% \) \
-append -resize 1920 ../$1.jpg
cd ..

if [[ -f "$1.jpg" ]]; then
echo "$1.jpg generated!"
rm -rf tmp
else
echo "Some error occurred!"
exit 1
fi
```

```bash
./get_thumbnail_imagemagick.sh <input_file>
```

- Concatenate

> Re-encoding

```bash
ffmpeg -i <input_file> -i <input_file> -filter_complex "[0:v:0][0:a:0][1:v:0][1:a:0]concat=n=2:v=1:a=1[v][a]" -map "[v]" -map "[a]" <output_file>
```

> No re-encoding

```bash
ffmpeg -f concat -safe 0 -i <(for f in ./*.mp4; do echo "file '$PWD/$f'"; done) -c copy <output_file>
```

- Trim

```bash
ffmpeg -i <input_file> -ss <start_time> -to <end_time> -c copy <output_file>
```

- Subtitle

> Soft

```bash
ffmpeg -i <input_file> -i <subtitle_file> -i <subtitle_file> -map 0:v -map 0:a -map 1 -map 2 -c copy -c:s mov_text -metadata:s:s:0 language=<lang> -metadata:s:s:1 language=<lang> <output_file>
```

> Hard

```bash
ffmpeg -i <input_file> -vf "ass=<ass_file>" <output_file>
```

```bash
ffmpeg -i <input_file> -vf "subtitles=<srt_file>" <output_file>
```
