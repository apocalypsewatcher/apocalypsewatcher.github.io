---
title: Salvando espaço convertendo FLAC para M4A.
date: 2025-10-27
author: Apocalypse Watcher
description: Shell script simples que converte flac para m4a economizando 80% de espaço em disco.
tags:
  - tech
  - flac
  - aac
  - m4a
  - linux
---
Shell script simples que converte FLAC para M4A economizando 80% de espaço em disco.

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

O script converte de FLAC para M4A sem perdas perceptíveis para a maioria dos casos. Uma pasta com 12GB foi reduzida
para 2.3GB um total de 80%.

