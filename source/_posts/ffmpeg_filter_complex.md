---
title: ffmpeg中的-filter_complex
date: 2023-04-08T21:38:06+08:00
toc: false
tags: 
categories: 
    - 应用软件
    - ffmpeg
---

## 合并

需要合并两个分辨率不同的mp4视频，所以稍微研究了下ffmpeg中的complex_filter的合并语法。

如果要是合并相同编码，分辨率，FPS的视频，那很简单，直接用`concat`文件协议就行：

```mylist.txt
# mylist.txt
file 'video1.mp4'
file 'video2.mp4'
```

（可以直接用`for f in *.mp4; do echo "file '$f'" >> mylist.txt; done`生成。）
然后

```bash
ffmpeg -f concat -safe 0 -i mylist.txt -c copy video.mp4
```

即可。

但是如果编码，分辨率，FPS不同，想用一行命令解决问题，就要用到`-filter_complex`参数，指定将哪个文件的哪个stream做什么样的变换。对于笔者的需求，将`video2.mp4`的分辨率转成和`video1.mp4`相同然后合并`video1.mp4`、`video2.mp4`，命令如下。

```bash
ffmpeg -i video1.mp4 -i video2.mp4 -filter_complex "[1:v]scale=640:480[v1];[0:v][0:a][v1][1:a]concat=n=2:v=1:a=1[outv][outa]" -map "[outv]" -map "[outa]" -c:v libx264 -crf 21 video.mp4
```

参数解释：

- `-i video1.mp4 -i video2.mp4`：输入视频，可以有多个；
- `-filter_complex`：后面跟complex filter表达式字符串（类似一种编程语言）
  - `[1:v]scale=640:480[v1]`: 将第二个输入文件的视频流分辨率转为640x480并用`[v1]`作为代号。`[1:v]`表示第一个输入视频文件的视频流，之所以这里是“1”是计算机界惯例编号从0开始。如果有多个视频流可以用`[1:v:0]`，`[1:v:1]`，……表示。
  - `[0:v][0:a][v1][1:a]concat=n=2:v=1:a=1[outv][outa]`表示将第一个输入文件的音视频流，转换后的第二个输入文件视频流，第二个输入文件的音频流连接起来并分别赋予代号`[outv]`和`[outa]`，`n=2`表示一共两个文件，`v=1`表示一共一个视频流，`a=1`表示一共一个音频流。
  - `-map "[outv]" -map "[outa]"`表示将`[outv]`和`[outa]`map为一个文件。
  - `-c:v libx264`指定视频流编码器，`-crf 21`是编码器参数
  - `video.mp4`指定输出文件名。

根据以上可以举一反三写出更复杂的filter表达式，具体的可用的filter可以参考[官方文档中的相关章节](https://www.ffmpeg.org/ffmpeg-filters.html)。


