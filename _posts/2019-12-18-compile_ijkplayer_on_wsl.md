---
layout: post
title:  "WSL环境下编译ijkplayer"
date:   2019-12-18 10:25:09 +0800--
categories: [Android]
tags:   [WSL, ijkplayer, android]
---

# 一、编译环境
## 1. 安装jdk
- 安装openjdk  
sudo apt-get install openjdk-8-jdk

- 配置环境变量  
vim /etc/profile，在文件末尾加上如下内容：
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
source /etc/profile，使环境变量生效

## 2. 安装android sdk
- 下载sdk  
地址：https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip

- 解压sdk-tools-linux-4333796.zip

- 下载platform-tools  
若sdk-tools-linux-4333796.zip解压后只有tools，则需要通过如下命令去下载platform-tools和具体的sdk版本：  
```
 # 列出已安装和可用的软件包
 ./tools/bin/sdkmanager --list
 # 下载具体的软件包
 ./tools/bin/sdkmanager "platform-tools" "platforms;android-29" 
```
有关sdkmanager命令详见[sdkmanager](https://developer.android.com/studio/command-line/sdkmanager "title")

- 配置sdk环境变量  
vim /etc/profile，在文件末尾加上如下内容：
```
export ANDROID_SDK=/home/chenming/android/android-sdk-linux
exprot PATH=$PATH:$ANDROID_SDK/tools:$ANDROID_SDK/platform-tools
```
source /etc/profile，使环境变量生效

## 3. 安装android ndk
- 下载ndk  
地址：https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip

- 解压android-ndk-r10e-linux-x86_64.zip

- 配置ndk环境变量  
vim /etc/profile，在文件末尾加上如下内容：
```
   export ANDROID_SDK=/home/chenming/android/android-ndk-r10e
   export PATH=$PATH:$ANDROID_NDK
```
source /etc/profile，使环境变量生效

<font color=red>注意：nkd的版本是r10e，如果是其他版本的ndk，在编译时可能会出现错误</font>  


## 4. 测试环境
- java -version  
```
chenming@DESKTOP-RN9FSGO:~$ java -version
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-8u222-b10-1ubuntu1~18.04.1-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
chenming@DESKTOP-RN9FSGO:~$
```
- adb --version  
```
chenming@DESKTOP-RN9FSGO:~$ adb --version
Android Debug Bridge version 1.0.41
Version 29.0.5-5949299
Installed as /home/chenming/android/android-sdk-linux/platform-tools/adb
chenming@DESKTOP-RN9FSGO:~$ 
```
- ndk-build --version  
```
chenming@DESKTOP-RN9FSGO:~$ ndk-build --version
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
This program built for x86_64-pc-linux-gnu
chenming@DESKTOP-RN9FSGO:~$     
```

# 二、编译步骤
## 1. 下载源码
- 下载ijkplayer源码  
```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android  
cd ijkplayer-android
```

- 下载编译所需的库  
进入ijkplayer-android目录，执行脚本`./init-android.sh`，如果需要支持https则还需执行脚本`./init-android-openssl.sh`

## 2. 编译
- 编译openssl(如果需要支持https)  
```
cd android/contrib
./compile-openssl.sh clean
./compile-openssl.sh all
```

- 编译ffmpeg  
```
cd android/contrib
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
```
`compile-ffmepg.sh`后面的参数代表编译哪个arm建构的版本，默认编译armv7a,编译选项，前面执行的`compile-openssl.sh`和后面执行的`compile-ijk.sh`后面的选项也是同样的道理。
```
armv5 armv7a arm64 x86 x86_64(指定编译哪个版本)
all32（所有32位处理器版本，包含armv5 armv7a x86）
all （所有通用版本，包含armv5 armv7a arm64 x86 x86_64）
clean （清除之前编译的缓存）
check (检测支持的版本)
```

- 编译ijkplayer
```
cd ..
./compile-ijk.sh all
```

# 三、遇到的问题  
- 执行./complie-ffmpeg.sh all  
```
build on Linux x86_64
ANDROID_NDK=/home/chenming/android/android-ndk-r14b
IJK_NDK_REL=14.1.3816874
NDKr14.1.3816874 detected
HOST_OS=linux
HOST_EXE=
HOST_ARCH=x86_64
HOST_TAG=linux-x86_64
HOST_NUM_CPUS=4
BUILD_NUM_CPUS=8
Auto-config: --arch=arm
ERROR: Failed to create toolchain.
```
原因：ndk版本不匹配，换成r10e就好了

- 执行./compile-openssl.sh all  
```
--------------------
[*] check openssl env
--------------------
tools/do-compile-openssl.sh: 157: export: (x86)/Common: bad variable name
```
原因：在do-complie-openssl.sh脚本文件中引用了$PATH，改PATH不仅包含了linux子系统的环境变量，也包含了windows系统的环境变量(无法被识别其中带空格的目录)，暂无解决办法
