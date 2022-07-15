# Android Performance Monitor

`Android Performance Monitor`或叫`BlockCanary`是[markzhai](https://github.com/markzhai)开源的一款Android平台的一个非侵入式的性能监控组件。

应用只需要实现一个抽象类，提供一些该组件需要的上下文环境，就可以在平时使用应用的时候检测主线程上的各种卡慢问题，并通过组件提供的各种信息分析出原因并进行修复。

[开原地址](https://github.com/markzhai/AndroidPerformanceMonitor)

[大佬的博客](https://blog.zhaiyifan.cn/)

[大佬自己的源码分析](http://blog.zhaiyifan.cn/2016/01/16/BlockCanaryTransparentPerformanceMonitor/)

## 如何监听卡顿的

是的，就是这个Printer - mLogging，它在每个message处理的前后被调用，而如果主线程卡住了，不就是在dispatchMessage里卡住了吗？

```java
public static void loop() {
    ...

    for (;;) {
        ...

        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        ...
    }
}
```

`BlockCanary.start`方法给`Looper`设置了一个`LooperMonitor`，LooperMonitor继承于`Printer`。

```java
public void start() {
    if (!mMonitorStarted) {
        mMonitorStarted = true;
        Looper.getMainLooper().setMessageLogging(mBlockCanaryCore.monitor);
    }
}
```

所以在Looper`完整执行一次消息`时候，判断`开始执行Message`和`结束执行Message`的耗时，就可以监听是否发生了卡顿。

```java
@Override
public void println(String x) {
    if (mStopWhenDebugging && Debug.isDebuggerConnected()) {
        return;
    }
    if (!mPrintingStarted) {
        mStartTimestamp = System.currentTimeMillis();
        mStartThreadTimestamp = SystemClock.currentThreadTimeMillis();
        mPrintingStarted = true;
        startDump();
    } else {
        final long endTime = System.currentTimeMillis();
        mPrintingStarted = false;
        if (isBlock(endTime)) {
            notifyBlockEvent(endTime);
        }
        stopDump();
    }
}
```
