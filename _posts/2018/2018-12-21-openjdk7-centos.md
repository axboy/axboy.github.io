---
layout: post
title: CentOS7下编译OpenJDK
description: CentOS7下编译OpenJDK
categories: java
tags: 
---

### 当前环境

```sh
root@test11:~# uname -av
Linux test11 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

root@test11:openjdk# java -version
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
```

### 相关下载

- [jdk7,不使用版本管理工具](http://jdk.java.net/java-se-ri/7)

### 环境配置(最终结果)

#### 新建文件 ~/.bash_jdk7，如下内容

```sh
#!/bin/bash

# 设置为false，避开javaws和浏览器插件部分的build
BUILD_DEPLOY=false

# 不输出安装包
BUILD_INSTALL=false

#语言选项
export LANG=C

# Bootstrap JDK的安装路径
# export ALT_BOOTDIR=/usr/local/java/jdk1.8.0_162
export ALT_BOOTDIR=/usr/local/java/jdk1.7.0_80

# 编译结果存放路径，代码路径
# export ALT_OUTPUTDIR=/data/openjdk/build

# 允许自动下载
export ALLOW_DOWNLOADS=true

# 并行线程数
export HOSTPOT_BUILD_JOBS=4
export ALT_PARALLEL_COMPILE_JOBS=4

# cups安装路径,这里未使用
# export #ALT_CUPS_HEADERS_PATH=/data/openjdk/cups-2.3b7

# 跳过对比旧版本镜像
export SKIP_COMPARE_IMAGES=true

# 使用预编译头文件
export USE_PRECOMPILED_HEADER=false

# 要编译的版本
export SKIP_DEBUG_BUILD=false
export SKIP_FASTDEBUG_BUILD=true
export DEBUG_NAME=debug

export COMPILER_WARNINGS_FATAL=false

# 去除这两个变量
unset JAVA_HOME
unset CLASSPATH
```

#### 在~/.bash_profile后追加以下内容

很多博客都是直接把相关配置直接添加到~/.bash_profile中，个人更推荐独立一个文件，不用的时候注释一行即可，不用大片删除

```sh
# 编译jdk7临时使用
if [ -f ~/.bash_jdk7 ]; then
    . ~/.bash_jdk7
fi
```

#### 检测环境和编译

先make sanity,有错按下面方式处理，再make

```
make sanity
make
```

### 错误处理

#### ERROR: You do not have access to valid Cups header files.

```
ERROR: You do not have access to valid Cups header files. 
Please check your access to 
/usr/include/cups/cups.h 
and/or check your value of ALT_CUPS_HEADERS_PATH, 
CUPS is frequently pre-installed on many systems, 
or may be downloaded from http://www.cups.org

找不到cups，安装
yum install cups-devel.x86_64
```

#### ERROR: FreeType version 2.3.0 or higher is required.

```
缺少freetype,安装
yum install freetype.x86_64 freetype-devel.x86_64
```

#### ERROR: You seem to not have installed ALSA 0.9.1 or higher.

```
ERROR: You seem to not have installed ALSA 0.9.1 or higher.
    Please install ALSA (drivers and lib). You can download the
    source distribution from http://www.alsa-project.org or go to
    http://www.freshrpms.net/docs/alsa/ for precompiled RPM packages.

以下命令处理
yum install alsa*
```

#### ERROR: The version of ant being used is older than the required version of '1.7.1'.

```
ERROR: The version of ant being used is older than
    the required version of '1.7.1'.
    The version of ant found was ''.

安装ant
yum install ant
```

#### build-bootstrap-javac

```
build-bootstrap-javac:
    [javac] Compiling 95 source files to /data/openjdk/build/linux-amd64-debug/langtools/build/bootstrap/classes
    [javac] /data/openjdk/langtools/src/share/classes/com/sun/tools/javac/comp/Resolve.java:2182: warning: [overrides] Class Resolve.InapplicableSymbolsError.Candidate overrides equals, but neither it nor any superclass overrides hashCode method
    [javac]         private class Candidate {
    [javac]                 ^
    [javac] error: warnings found and -Werror specified
    [javac] 1 error
    [javac] 1 warning

BUILD FAILED
/data/openjdk/langtools/make/build.xml:452: The following error occurred while executing this line:
/data/openjdk/langtools/make/build.xml:795: Compile failed; see the compiler error output for details.

换一个jdk，安装jdk7
jdk7链接不好找，下载链接如下
http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u80-oth-JPR
```

#### cannot find -lstdc++

```
Linking vm...
/usr/bin/ld: cannot find -lstdc++
collect2: error: ld returned 1 exit status
/usr/bin/chcon: cannot access 'libjvm.so': No such file or directory
ERROR: Cannot chcon libjvm.so
/usr/bin/objcopy --only-keep-debug libjvm.so libjvm.debuginfo
/usr/bin/objcopy: 'libjvm.so': No such file

yum install glibc-static libstdc++-static
```

#### zip: Command not found

```
make[7]: zip: Command not found
make[7]: *** [libjvm.so] Error 127

yum install zip
```

#### time is more than 10 years from present: 1136059200000

```
Error: time is more than 10 years from present: 1136059200000

vi /data/openjdk/jdk/src/share/classes/java/util/CurrencyData.properties
搜索00，修改时间
```

#### libXt-devel

```
../../../src/solaris/native/sun/awt/awt.h:38:27: fatal error: X11/Intrinsic.h: No such file or directory
#include <X11/Intrinsic.h>
                        ^
compilation terminated.

yum install libXt-devel
```

#### libXext-devel

```
In file included from ../../../src/share/native/sun/awt/splashscreen/splashscreen_impl.h:29:0,
                from ../../../src/share/native/sun/awt/splashscreen/java_awt_SplashScreen.c:26:
../../../src/solaris/native/sun/awt/splashscreen/splashscreen_config.h:33:34: fatal error: X11/extensions/shape.h: No such file or directory
#include <X11/extensions/shape.h>

yum install libXext-devel
```

#### libXrender-devel

```
In file included from /data/openjdk/build/linux-amd64/../linux-amd64-debug/gensrc/sun/awt/X11/generator/sizer.64.c:11:0:
../../../src/solaris/native/sun/awt/awt_p.h:51:36: fatal error: X11/extensions/Xrender.h: No such file or directory
#include <X11/extensions/Xrender.h>
yum install libXrender-devel
```

#### libXtst-devel

```
../../../src/solaris/native/sun/xawt/XToolkit.c:48:34: fatal error: X11/extensions/XTest.h: No such file or directory
#include <X11/extensions/XTest.h>
yum install libXtst-devel
```

#### cc1: all warnings being treated as errors

```
../../../../../src/solaris/native/sun/nio/ch/SctpNet.c: In function 'JNI_OnLoad':
../../../../../src/solaris/native/sun/nio/ch/SctpNet.c:47:12: error: unused parameter 'vm' [-Werror=unused-parameter]
(JavaVM *vm, void *reserved) {
...
make[8]: *** [/data/openjdk/build/linux-amd64/../linux-amd64-debug/tmp/sun/com.sun.nio.sctp/sctp/obj64_g/SctpChannelImpl.o] Error 1
make[8]: *** Waiting for unfinished jobs....
cc1: all warnings being treated as errors

搜索包含Werror的Makefile文件，把如下四个文件的Werror去了

find -name "Makefile" | xargs grep "Werror"
./jdk/make/java/nio/Makefile:OTHER_JAVACFLAGS += -Xmaxwarns 1000 -Xlint:serial -Werror
./jdk/make/java/sun_nio/Makefile:OTHER_JAVACFLAGS += -Xlint:serial,-deprecation -Werror
./jdk/make/sun/native2ascii/Makefile:OTHER_JAVACFLAGS += -Xlint:serial -Werror
./jdk/make/sun/nio/cs/Makefile:OTHER_JAVACFLAGS += -Xlint:serial,-deprecation -Werror

发现还报错，放宽条件继续搜，把jdk/make/common/Defs-linux.gmk中的Werror也去掉
```

### 测试

```
#-- Build times ----------
Target debug_build
Start 2018-12-21 21:01:27
End   2018-12-21 21:10:19
00:00:55 corba
00:02:57 hotspot
00:00:08 jaxp
00:00:09 jaxws
00:04:28 jdk
00:00:15 langtools
00:08:52 TOTAL
-------------------------

build/linux-amd64/bin/java -version
openjdk version "1.7.0-internal-debug"
OpenJDK Runtime Environment (build 1.7.0-internal-debug-root_2018_12_21_20_59-b00)
OpenJDK 64-Bit Server VM (build 24.75-b04-jvmg, mixed mode)
```
