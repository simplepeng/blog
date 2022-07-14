# Matrix

Matrix是一款微信研发并日常使用的应用性能接入框架，支持iOS, macOS和Android。 

Matrix 通过接入各种性能监控方案，对性能监控项的异常数据进行采集和分析，输出相应的问题分析、定位与优化建议，从而帮助开发者开发出更高质量的应用。

[开源地址](https://github.com/Tencent/matrix)

## Matrix for Android

Matrix-android 当前监控范围包括：应用安装包大小，帧率变化，启动耗时，卡顿，慢方法，SQLite 操作优化，文件读写，内存泄漏等等。

* APK Checker: 针对 APK 安装包的分析检测工具，根据一系列设定好的规则，检测 APK 是否存在特定的问题，并输出较为详细的检测结果报告，用于分析排查问题以及版本追踪
* Resource Canary: 基于 WeakReference 的特性和 Square Haha 库开发的 Activity 泄漏和 Bitmap 重复创建检测工具
* Trace Canary: 监控ANR、界面流畅性、启动耗时、页面切换耗时、慢函数及卡顿等问题
* SQLite Lint: 按官方最佳实践自动化检测 SQLite 语句的使用质量
* IO Canary: 检测文件 IO 问题，包括：文件 IO 监控和 Closeable Leak 监控
* Battery Canary: 监控 App 活跃线程（待机状态 & 前台 Loop 监控）、ASM 调用 (WakeLock/Alarm/Gps/Wifi/Bluetooth 等传感器)、 后台流量 (Wifi/移动网络)等 Battery Historian 统计 App 耗电的数据
* MemGuard：检测堆内存访问越界、使用释放后的内存、重复释放等问题
