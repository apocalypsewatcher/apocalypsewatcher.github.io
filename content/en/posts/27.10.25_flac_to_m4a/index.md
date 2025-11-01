---
Title: Saving Space by Converting FLAC to M4A
Date: 2025-10-27
Author: Apocalypse Watcher
Description: Simple shell script that converts FLAC to M4A, saving 80% of disk space.
tags:
  - tech
  - flac
  - aac
  - m4a
  - linux
---
A simple shell script that converts FLAC to M4A, saving 80% of disk space.

```sh
flactom4a() {
  thumbnail() {
    ffprobe -v error -select_streams v -show_entries stream=codec_type -of csv=p=0 "$1" 2>/dev/null | grep -q .
  }
  if dpkg-query -W aac-enc >/dev/null 2>&1; then
    for f in *.flac; do
      if thumbnail "$f"; then
        ffmpeg -v warning -stats -i "$f" -map 0:a -map 0:v \
          -filter:v "format=yuvj420p" -q:v 3 \
          -c:a libfdk_aac -vbr 5 \
          -c:v mjpeg \
          -ac 2 -ar 44100 -map_metadata 0 -movflags +faststart \
          "${f%.flac}.m4a"
      else
        ffmpeg -v warning -stats -i "$f" -map 0:a \
          -c:a libfdk_aac -vbr 5 \
          -ac 2 -ar 44100 -map_metadata 0 -movflags +faststart \
          "${f%.flac}.m4a"
      fi
    done
  else
    for f in *.flac; do
      if thumbnail "$f"; then
        ffmpeg -v warning -stats -i "$f" -map 0:a -map 0:v \
          -filter:v "scale=600:-1,format=yuvj420p" -q:v 3 \
          -c:a aac -b:a 256k \
          -c:v mjpeg \
          -ac 2 -ar 44100 -map_metadata 0 -movflags +faststart \
          "${f%.flac}.m4a"
      else
        ffmpeg -v warning -stats -i "$f" -map 0:a \
          -c:a aac -b:a 256k \
          -ac 2 -ar 44100 -map_metadata 0 -movflags +faststart \
          "${f%.flac}.m4a"
      fi
    done
  fi
}
```

The script converts FLAC to M4A with no perceptible loss in most cases. A folder of 12 GB was reduced to 2.3 GB
a total of 80% saved.
