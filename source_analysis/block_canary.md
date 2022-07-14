# Android Performance Monitor

`Android Performance Monitor`或叫`BlockCanary`是[markzhai](https://github.com/markzhai)开源的一款Android平台的一个非侵入式的性能监控组件。

应用只需要实现一个抽象类，提供一些该组件需要的上下文环境，就可以在平时使用应用的时候检测主线程上的各种卡慢问题，并通过组件提供的各种信息分析出原因并进行修复。

[开原地址](https://github.com/markzhai/AndroidPerformanceMonitor)

[大佬的博客](https://blog.zhaiyifan.cn/)

[大佬自己的源码分析](http://blog.zhaiyifan.cn/2016/01/16/BlockCanaryTransparentPerformanceMonitor/)

## 如何监听卡顿的

