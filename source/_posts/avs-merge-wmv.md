---
title: AVS合并视频文件
toc: true
date: 2018-10-02 14:55:46
categories:
- 视频压制
---

　　常用的视频压制软件，如MeGUI、小丸工具箱等都不支持直接合并视频。所以只能通过AVS脚本实现。但直接用工具生成的脚本压制后会有音画不同步问题。这里分享一种网上找到的办法，可以基本保证合并并压制出没有明显瑕疵的视频。
　　基本思路就是音画分别压制，视频部分用MeGUI自带的Avs Script Creator，生成后合并到一个文件里：
```
LoadPlugin("E:\MediaTools\MeGUI-2715-32\tools\lsmash\LSMASHSource.dll")
LoadPlugin("E:\MediaTools\MeGUI-2715-32\tools\ffms\ffms2.dll")
A=FFVideoSource("F:\JDownloader\Downloads\A.wmv", fpsnum=30, fpsden=1, threads=1)
B=FFVideoSource("F:\JDownloader\Downloads\B.wmv", fpsnum=30, fpsden=1, threads=1)
C=FFVideoSource("F:\JDownloader\Downloads\C.wmv", fpsnum=30, fpsden=1, threads=1)
D=FFVideoSource("F:\JDownloader\Downloads\D.wmv", fpsnum=30, fpsden=1, threads=1)
A+B+C+D
#deinterlace
#crop

LanczosResize(848,480) # Lanczos (Sharp)
#denoise
```
音频部分直接用DirectShowSource，并强制帧率，
```
A = DirectShowSource("F:\JDownloader\Downloads\A.wmv", fps=30.000, audio=true, convertfps=true)
B = DirectShowSource("F:\JDownloader\Downloads\B.wmv", fps=30.000, audio=true, convertfps=true)
C = DirectShowSource("F:\JDownloader\Downloads\C.wmv", fps=30.000, audio=true, convertfps=true)
D = DirectShowSource("F:\JDownloader\Downloads\D.wmv", fps=30.000, audio=true, convertfps=true)
A+B+C+D
```
有个问题是MeGUI的x264似乎有些问题，所以这里视频要用小丸压，音频用MeGUI。最后用MeGUI的MP4Box做混流。
　　不是很完美的解决方案，有些麻烦，不过可以达到目的。