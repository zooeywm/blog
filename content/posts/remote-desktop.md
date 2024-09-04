+++
title = 'Remote Desktop'
date = 2024-09-04T20:38:39+08:00
lastUpdate = 2024-09-04T20:38:39+08:00
summary = "Remote Desktop Essay"
tags = ["Essay"]
+++

# 技术栈

- slint: GUI
- ffmpeg: decoder
- ffmpeg: render
- winit: windows manage

# 性能调优记录

技术方案 1：winit启动窗口->读取视频文件->ffmpeg 解码->ffmpeg 软件渲染->交给 slint->交给 skia -> 调用图形 api 显示

评价标准：在 debug 模式下，从程序启动开始计时，记录每一帧从渲染完成到提交给 skia 的当前经过的时间，并且记录cpu、内存、用时，cpu、内存占用越少，用时越少性能越高

控制变量不变：winit，读取视频文件，ffmpeg 解码方式, 解码在单独线程, slint, skia, 图形api

变量：

1. 软件渲染调度方式（因为用不到异步 io 或者时间处理，所以不考虑异步）：
    a. 本线程同步: ✅
    b. 新线程同步
2. 软件渲染结果展示方式：
    a. 单独线程使用管道，异步多线程接收到消息再调用 slint 回调
    b. 直接渲染结束同步调用 slint 回调: ✅

结论：使用本线程同步渲染并直接调用 slint 回调，比花里胡哨的开线程性能要高一倍左右。
