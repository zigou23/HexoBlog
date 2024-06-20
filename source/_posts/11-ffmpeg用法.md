---
title: ffmpeg用法
date: 2023-07-16 14:00:00
categories: 
- ffmpeg
tags:
- ffmpeg
- 压缩
---

# ffmpeg用法（仅自己方便使用

1. 视频快速剪切一小段（会出现问题

```
ffmpeg -ss 10 -t 15 -accurate_seek -i test.mp4 -codec copy -avoid_negative_ts 1 cut.mp4
怕出问题删掉-codec copy（较慢
ffmpeg -ss 10 -t 15 -accurate_seek -i test.mp4 cut.mp4
ffmpeg -ss 10 -t 15 -i test.mp4 -c:v libx264 -c:a aac -strict experimental -b:a 98k cut.mp4 -y
```

```
-ss:开始时间	-t:持续时间
时间格式HOURS:MM:SS.MICROSECONDS
```



2. 手机ffmpeg

```makefile
# 裁剪到QQ语音(群1M以内)
-ss 00:00:00 -t 00:02:00 -vn -c:a libmp3lame -ab 64k -ar 44100 -f mp3
```

