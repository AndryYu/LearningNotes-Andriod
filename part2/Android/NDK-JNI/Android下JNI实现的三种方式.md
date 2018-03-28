## Android下JNI实现的三种方式
安装ndk插件，google ndk-build命令在r13后默认使用Clang，并将在后续版本中移除GCC，其编译速度更快、编译产出更小、出错提示更友好。三种实现方式如下：
* 徒手编写Android.mk然后ndk-build编译
* 通过配置AS中的build.gradle来编译
* 通过AS的CMake插件编译

### 1.徒手编写Android.mk然后ndk-build编译
这种编译其实是用make工具来实现，在linux徒手写并编译过C的应该很清楚，通过编写makefile，然后再用make编译已经比不停的用gcc命令逐个编译要方便很多。但是makefile的编写还是比较不方便，通过ndk工具来帮助我们实现makefile会简单很多。



















### 2.通过配置AS中的build.gradle来编译
这种方式比上一种又简化了很多，无需自己编写Android.mk，其他原理是一样的。






















### 3.通过AS的CMake插件编译
使用CMake插件要求Android Studio版本必须在2.2及以上。





#### CMakeLists.txt讲解
```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```
* add_library:我们需要在里面指定三个东西滴，首先给lib取一个名字，然后指定作为动态库还是静态库，最后指定源文件或者库。<br/>
  使用add_library()向你的CMake构建脚本添加源文件或库时，Android studio还会在你同步项目后，在Project视图下显示关联的标头文件。不过，为了确保CMake可以在编译时定位你的标头文件，你需要将include_directories()命令添加到CMake构建脚本中并指定标头的路径：
  ```
  add_library(...)

  # Specifies a path to native header files.
  include_directories(src/main/cpp/include/)
  ```
* find_library:将find_library()命令添加到你的CMake构建脚本中以定位NDK库，并将其路径存储为一个变量。你可以使用此变量在构建脚本的其他部分引用NDK库。
* target_link_libraries:指定要关联到原生库的库，第一个自然是我们add_libray里面指定的库文字native-lib库，然后可以看到${log-lib}，也就是引用了find_library里面定义的日志库。






** 参考文献**<br/>
[Android下玩JNI的新老三种姿势](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650823124&idx=1&sn=7bb627c2bcf6447f9bf7824bef30c387&chksm=80b78d4ab7c0045c811126d8e4a5132d71d4416c7d842c9f3e8dd077248c62350d3b9fa08a23&mpshare=1&scene=23&srcid=0603fbkWZY8oQcompGY4SQV6#rd)
