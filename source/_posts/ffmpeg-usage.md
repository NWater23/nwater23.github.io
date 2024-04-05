---
title: ffmpeg usage
date: 2024-04-05 20:27:13
updated: 2024-04-05 20:27:13
tags:
- ffmpeg
categories:
- 笔记
---

今天又是被ffmpeg折腾得死去活来的一天，为了避免以后浪费时间，有必要记录一下。

### 硬件加速
`-hwaccel <method>` 可以用 `-hwaccels` 查看可用 method

`-c:v <encoder>` encoder 必须为硬件支持的编解码器：`h264_qsv`, `hevc_qsv`

### 质量控制

### 视频剪辑
`-ss <hh:mm:ss>` `-to <hh:mm:ss>` 两个开关控制起始时间和结束时间。

`-f concat -i <concat_list_file>` 拼接视频，concat_list_file 中需要预先定义好需要的视频文件，可以这样生成：
```bash
for f in *.mp4; do echo "file '$f'" >> mylist.txt; done
```

### 画面编辑

#### 压制硬字幕（嵌入画面中）

`-vf "subtitles=example_subtitle.vtt"`

#### 马赛克

```bash
ffmpeg -i input.mp4 -i mask.png -filter_complex "[0:v][1:v]alphamerge,avgblur=128[alf];[0:v][alf]overlay[v]" -map "[v]" -map 0:a -c:v h264_qsv -c:a copy -movflags +faststart output.mp4
```

`avgblur=<number>` 数字越大越模糊。

`-i mask.png` 使用 `mask.png` 作为遮罩，这张图片需要与视频画面大小一致，白色像素对应打码区域，黑色像素对应保持原样。 [^mask-png]

[^mask-png]: https://superuser.com/a/901705