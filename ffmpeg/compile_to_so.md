# 编译成动态库

## 环境

* macOS High Sierra
* FFmpeg 4.3
* android-ndk-r21b

## 编译so库

下载FFmpeg4.3源代码，进入源码目录创建`build_android.sh`脚本，ffmpeg从4.0起新增了`target-os=android`，所以不用再修改`configure`文件。

**注意：**

* `ndk-17`以前内置的编译器是`gcc`，而新版的ndk已经用`clang`替代了`gcc`编译器，所以使用`-cc`和`-cxx`指令时要特别注意自己要使用的编译器是`gcc`还是`clang`。

* 还有个我遇到的问题就是在`ndk-r17c`和`ndk-18b`中的`toolchains/llvm/prebuilt/darwin-x86_64/bin`中没有`clang`编译工具集，而是在上层目录中。
* 只有`armv7-a`的执行文件中间有`eabi`结尾，其他没有。例：`armv7a-linux-androideabi21-clang`，`x86_64-linux-android21-clang`。如果报错`not found xxx file`的错误，就自行到相应目录查看。

```shell
#!/bin/bash

# ndk路径
NDK=/Users/chenpeng/Desktop/work_space/ndk/android-ndk-r21b
# 编译工具链目录，ndk17版本以上用的是clang，以下是gcc
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/darwin-x86_64
# 版本号
API=21
# 交叉编译树的根目录(查找相应头文件和库用)
SYSROOT="${TOOLCHAIN}/sysroot"

# armv7-a
OUTPUT_FOLDER="armeabi-v7a"
ARCH="arm"
CPU="armv7-a"
TOOL_CPU_NAME=armv7a
TOOL_PREFIX="$TOOLCHAIN/bin/${TOOL_CPU_NAME}-linux-androideabi"
OPTIMIZE_CFLAGS="-march=$CPU"

# arm64-v8a，这个指令集最低支持api21
# OUTPUT_FOLDER="arm64-v8a"
# ARCH="aarch64"
# CPU="armv8-a"
# TOOL_CPU_NAME=aarch64
# TOOL_PREFIX="$TOOLCHAIN/bin/${TOOL_CPU_NAME}-linux-android"
# OPTIMIZE_CFLAGS="-march=$CPU"

# x86
# OUTPUT_FOLDER="x86"
# ARCH="x86"
# CPU="x86"
# TOOL_CPU_NAME="i686"
# TOOL_PREFIX="$TOOLCHAIN/bin/${TOOL_CPU_NAME}-linux-android"
# OPTIMIZE_CFLAGS="-march=i686 -mtune=intel -mssse3 -mfpmath=sse -m32"

# x86_64，这个指令集最低支持api21
# OUTPUT_FOLDER="x86_64"
# ARCH="x86_64"
# CPU="x86_64"
# TOOL_CPU_NAME="x86_64"
# TOOL_PREFIX="$TOOLCHAIN/bin/${TOOL_CPU_NAME}-linux-android"
# OPTIMIZE_CFLAGS="-march=$CPU"

# 输出目录
PREFIX="${PWD}/android/$OUTPUT_FOLDER"
# so的输出目录， --libdir=$LIB_DIR 可以不用指定，默认会生成在$PREFIX/lib目录中
#LIB_DIR="${PWD}/android/libs/$OUTPUT_FOLDER"
# 编译器
CC="${TOOL_PREFIX}${API}-clang"
CXX="${TOOL_PREFIX}${API}-clang++"

# 定义执行configure的shell方法
function build_android() {
    ./configure \
        --prefix=$PREFIX \
        --enable-shared \
        --disable-static \
        --enable-jni \
        --disable-doc \
        --disable-programs \
        --disable-symver \
        --target-os=android \
        --arch=$ARCH \
        --cpu=$CPU \
        --cc=$CC \
        --cxx=$CXX \
        --enable-cross-compile \
        --sysroot=$SYSROOT \
        --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \
        --extra-ldflags="" \
        --disable-asm \
        $COMMON_FF_CFG_FLAGS
    make clean
    make -j8
    make install
}
build_android
```

更多构建参数可以使用`./configure -h`参看

shell脚本语言不熟的可以查看我的[shell学习笔记](https://github.com/simplepeng/KeepLearning/tree/master/Shell)

执行`build_android.sh`脚本，等待脚本执行完成，执行过程可能会遇到缺少组件的问题，按需解决。

```shell
# 如遇到permission denied，请chmod 777 xx.sh
sh build_android.sh
```

生成文件目录如下：

![](https://raw.githubusercontent.com/simplepeng/ImageRepo/master/android/ffmpeg_1_1.png)

## 创建一个新的Android Cmake项目

将动态库放入`libs/armeabi-v7a`文件夹，将头文件方法`cpp`目录

![](https://raw.githubusercontent.com/simplepeng/ImageRepo/master/android/ffmpeg_1_2.png)

新增`ffmpeg_utils.cpp`类，配置`build.gradle`，配置`CMakeLists.txt`

```c++
#include <jni.h>

//这里要注意，因为这是个c++文件，必须把头文件引用放到extern "C"中
//不然一直报undefined的错误，坑了很久，以前我都是写c
extern "C" {
#include <libavutil/avutil.h>

JNIEXPORT jstring JNICALL
Java_demo_simple_example_1ffmpeg_MainActivity_getVersion(JNIEnv *env, jclass clazz) {
    const char *version = av_version_info();
    return env->NewStringUTF(version);
}

}
```

```groovy
android {
    compileSdkVersion 29
    buildToolsVersion "30.0.0"

    defaultConfig {
        applicationId "demo.simple.example_ffmpeg"
        minSdkVersion 21
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"

        //ndk so架构的选择
        externalNativeBuild {
            ndk {
                abiFilters 'armeabi-v7a'
            }
        }

        //so查找路径
        sourceSets {
            main {
                jniLibs.srcDirs = ['libs']
            }
        }
    }

    //cmake文件的查找路径
    externalNativeBuild {
        cmake {
            path file('CMakeLists.txt')
        }
    }
}
```

```cmake
# 设置构建本机库文件所需的 CMake的最小版本
cmake_minimum_required(VERSION 3.4.1)

#添加头文件的搜索路径
include_directories(src/main/cpp/include/)

# 添加自己写的 C/C++源文件
add_library(utils #so名称
        SHARED #动态库
        src/main/cpp/ffmpeg_utils.cpp
        )

# 添加外部的库(可以是动态库或静态库)，这里只引入了avutil
set(LIBS_DIR ${CMAKE_SOURCE_DIR}/libs/${ANDROID_ABI})
add_library(
        avutil
        SHARED
        IMPORTED)
set_target_properties(
        avutil
        PROPERTIES IMPORTED_LOCATION
        ${LIBS_DIR}/libavutil.so)

#  依赖 NDK中自带的log库
find_library(log-lib log)

#  链接库
target_link_libraries(
        utils
        avutil
        ${log-lib})
```

## 加载动态库，获取FFmpeg的版本号

```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView tvVersion = findViewById(R.id.tvVersion);
        String version = String.format("ffmpeg version == %s", getVersion());
        tvVersion.setText(version);

        Log.d(TAG, version);
    }

    static {
        System.loadLibrary("utils");
        System.loadLibrary("avutil");
    }

    public static native String getVersion();
}
```

> 输出结果：
>
> ffmpeg version == 4.3

## 简写导入动态库的cmake语句

上面`CmakeList`引入`avutil`使用了`add_library`和`set_target_properties`，如果同时引用很多动态库，那就要写很多的重复配置，我们完全可以使用下面的方式简写配置。或者也可以将所有动态库合并成一个动态库。

```cmake
# 去掉`add_library`和`set_target_properties`的相关配置

#设置查找动态库位置
set(LINK_DIR ${CMAKE_SOURCE_DIR}/libs/${CMAKE_ANDROID_ARCH_ABI})
link_directories(${LINK_DIR})
#找到所有的so库，存放在全局变量SO_DIR中
file(GLOB SO_DIR ${LINK_DIR}/*.so)

#  链接库
target_link_libraries(
        utils
       ${SO_DIR}
        ${log-lib})
```

## 源码地址

[example_ffmpeg](https://github.com/simplepeng/AndroidExamples/tree/master/example_ffmpeg)
