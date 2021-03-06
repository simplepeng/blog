# 获取视频封面图Bitmap

这是学习ffmpeg的第二篇博客，主要是使用ffmpeg获取本地视频文件的第一帧数据转换为Bitmap，然后抛给上层ImageView显示。

大致流程可以分为：

1. 传入视频文件路径，解封装
2. 找到视频流，从流中找到解码器
3. 打开解码器，读取第一个完整的AVFrame帧
4. 创建bitmap，使用libyuv将yuv转为argb关联给bitmap显示
5. 释放资源

## 先定义jni函数

```java
public static native Bitmap getCover(String path);
```

```c++
JNIEXPORT jobject JNICALL
Java_demo_simple_example_1ffmpeg_MainActivity_getCover(JNIEnv *env,
                                                       jclass clazz,
                                                       jstring path) {
  const char *_path = env->GetStringUTFChars(path, JNI_FALSE);
  int ret = -1;
}
```

## 打开视频文件，解封装

```c++
//封装格式上下文    
AVFormatContext *ifmt_ctx = NULL;
//打开输入源
ret = avformat_open_input(&ifmt_ctx, _path, 0, 0);
if (ret < 0) {
    logDebug("解封装失败 -- %s", av_err2str(ret));
    return nullptr;
}
```

使用`avformat_open_input()`函数从输入文件中找到格式化I/O上下文`AVFormatContext`结构体，如果是编码要新建`AVFormatContext`要使用`avformat_alloc_context() `函数。

使用`avformat_open_input()`务必记得在程序执行完成后调用`avformat_close_input()`释放资源，并且该方法有一个`int`的返回值，`0`表示执行成功，`负数`表示执行失败，我们也可以用`av_err2str()`函数获取执行函数失败的详细日志，该函数定义在`libavutil/error.h`头文件中。

这里有个小技巧，ffmpeg大多有返回值函数，`大于等于0`都为成功，小于0都为执行失败。

## 找到流，从流的数组中找到视频流

```c++
ret = avformat_find_stream_info(ifmt_ctx, 0);
int video_stream_index = -1;
AVStream *pStream = NULL;
AVCodecParameters *codecpar = NULL;
//找出视频流
for (int i = 0; i < ifmt_ctx->nb_streams; ++i) {
    pStream = ifmt_ctx->streams[i];
    if (pStream->codecpar->codec_type == AVMEDIA_TYPE_VIDEO) {
        codecpar = pStream->codecpar;
        video_stream_index = i;
    }
}
```

`avformat_find_stream_info()`函数从媒体文件的数据包中获取流信息赋值给`AVFormatContext`。

`AVFormatContext`结构体中的`streams`字段包含了媒体文件中所以的流信息，包含`视频流`，`音频流`，`字幕流`等。

`AVCodecParameters`结构体描述了被编码后的流的属性。

谢谢大佬提点，这里找流的index其实可以使用`av_find_best_stream()`函数直接找到最合适的那个流index。

```c++
//找到最合适的流
int stream_index = av_find_best_stream(ifmt_ctx, AVMEDIA_TYPE_VIDEO, -1, -1, NULL, 0);
```

## 找到解码器，申请编解码器上下文

```c++
logDebug("解码器 == %s", avcodec_get_name(codecpar->codec_id));
AVCodec *codec = avcodec_find_decoder(codecpar->codec_id);
//申请编解码器上下文
AVCodecContext *codec_ctx = avcodec_alloc_context3(codec);
```

## 打开编解码器，获取首帧

```c++
//拷贝parameters 到 编解码器的context
ret = avcodec_parameters_to_context(codec_ctx, codecpar);
//打开编解码器
ret = avcodec_open2(codec_ctx, codec, NULL);
//申请一个帧结构体
AVFrame *pFrame = av_frame_alloc();

int frameFinished;
while (av_read_frame(ifmt_ctx, &pkg) >= 0) {
    if (pkg.stream_index != video_stream_index) {
        continue;
    }
    ret = avcodec_decode_video2(codec_ctx, pFrame, &frameFinished, &pkg);
    if (!frameFinished)
        continue;
}
```

`av_read_frame()`函数读取视频流信息，并将其存放到 AVPacket 结构的 pkt 变量中，这里我们只需分配 AVPacket 结构体的内存，数据`（pkt->data）`的数据则由 FFmpeg 在其内部自动分配，不过使用完毕后，要调用` av_packet_unref()`函数释放。

`av_read_frame()`函数的返回值如果`小于0`代表发生了错误或者是读到了文件的末尾。

`avcodec_decode_video()`函数可以将 packet 转换成 frame，不过，解码一个 packet 不一定能够获得 frame 的全部信息，所以需要借助` frameFinished` 标志位用于判断这一过程。其实新版已经使用`avcodec_send_packet()` 和`avcodec_receive_frame()`代替了这个函数，但是这里用下也没关系。

`frameFinished`参数`=0`时表示没有帧可以解压缩了，反之不为0时就还可以继续解压缩。

## 创建Bitmap

```c++
jobject createBitmap(JNIEnv *env,
                     int width, int height) {

    jclass bitmapCls = env->FindClass("android/graphics/Bitmap");
    jmethodID createBitmapFunction = env->GetStaticMethodID(bitmapCls,
                                                            "createBitmap",
                                                            "(IILandroid/graphics/Bitmap$Config;)Landroid/graphics/Bitmap;");
    jstring configName = env->NewStringUTF("ARGB_8888");
    jclass bitmapConfigClass = env->FindClass("android/graphics/Bitmap$Config");
    jmethodID valueOfBitmapConfigFunction = env->GetStaticMethodID(bitmapConfigClass,
                                                                   "valueOf",
                                                                   "(Ljava/lang/String;)Landroid/graphics/Bitmap$Config;");

    jobject bitmapConfig = env->CallStaticObjectMethod(bitmapConfigClass,
                                                       valueOfBitmapConfigFunction,
                                                       configName);

    jobject newBitmap = env->CallStaticObjectMethod(bitmapCls,
                                                    createBitmapFunction,
                                                    width, height,
                                                    bitmapConfig);

    return newBitmap;
}
```

常规操作，在native层调用java层的方法，jni不熟的可以查看[我的JNI学习笔记](../ndk/learn_jni.md)

## 使用libyuv写入rgb像素信息

先使用`bitmap.h`头文件中`AndroidBitmap_lockPixels()`的函数绑定像素指针的地址，使用libyuv中的`I420ToABGR()`函数将`yuv420p`转换为`argb`，记得最后使用`AndroidBitmap_unlockPixels()`函数回收Bitmap。

```c++
jobject bmp;
bmp = createBitmap(env, codec_ctx->width, codec_ctx->height);

void *addr_pixels;
ret = AndroidBitmap_lockPixels(env, bmp, &addr_pixels);

//yuv420p to argb
int linesize = pFrame->width * 4;
libyuv::I420ToABGR(pFrame->data[0], pFrame->linesize[0], // Y
                   pFrame->data[1], pFrame->linesize[1], // U
                   pFrame->data[2], pFrame->linesize[2], // V
                   (uint8_t *) addr_pixels, linesize,  // RGBA
                   pFrame->width, pFrame->height);
```

上面的`lineSize`计算规则为：一个像素点有4个字节，所以要宽度x4

Yuv420p转ARGB的函数名是`I420ToABGR`，并不是`I420ToARGB`。我坑，之前手打代码过快，导致生成的Bitmap颜色显示一直都不对，找了好久都没发现错误在哪里，还是百度到了一篇博主也是犯了同样的错误，这就是人的固化思维啊！

## 释放资源

```c++
av_packet_unref(&pkg);
AndroidBitmap_unlockPixels(env, bmp);
av_free(pFrame);
avcodec_close(codec_ctx);
avformat_close_input(&ifmt_ctx);
env->ReleaseStringUTFChars(path, _path);
```

## Activity中的代码

```java
 String path = Environment.getExternalStorageDirectory().getAbsolutePath() + File.separator
         + "get_cover.mp4";
 Bitmap bitmap = getCover(path);

 Log.d(TAG, "bitmap width == " + bitmap.getWidth());
 Log.d(TAG, "bitmap height == " + bitmap.getHeight());
 Log.d(TAG, "bitmap config == " + bitmap.getConfig().name());
 Log.d(TAG, "bitmap byteCount == " + bitmap.getByteCount());

 ImageView ivCover = findViewById(R.id.ivCover);
 ivCover.setImageBitmap(bitmap);
```

## Log输出和界面显示

> demo.simple.example_ffmpeg D/MainActivity: bitmap width == 1080
> 
> demo.simple.example_ffmpeg D/MainActivity: bitmap height == 1920
> 
> demo.simple.example_ffmpeg D/MainActivity: bitmap config == ARGB_8888
> 
> demo.simple.example_ffmpeg D/MainActivity: bitmap byteCount == 8294400

![界面实现](imgs/ffmpeg_1_1.png)

## 完整代码

[example_ffmpeg](https://github.com/simplepeng/AndroidExamples/tree/master/example_ffmpeg)
