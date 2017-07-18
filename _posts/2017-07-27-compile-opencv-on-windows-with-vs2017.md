---
title: 在 Windows 用 VS2017 编译 OpenCV
date: 2017-07-27
categories:
  - devops
tags:
  -opencv
  -vs2017
---

最先开始考虑用 Qt 安装包自带的 mingw32 来编译, 结果发现满满都是坑, 譬如:

1. <https://github.com/opencv/opencv/issues/7652>
1. <http://answers.opencv.org/question/62580/not-able-to-build-opencv3-rc1-with-debug-build-type/>
1. <http://blog.csdn.net/lucksis/article/details/60580861>

与其又是改源文件, 又是加标记之类的, 还不如老老实实上 Visual Studio 2017 好了, 在
Windows 平台, 还是微软的最稳妥. 想必在 Linux 上进行编译, 也不会是这番景象.

这篇文章恰好是<[Learning OpenCV][0]>的第一章[习题][1].

## 下载源代码

本着学习 OpenCV 的目的, 我直接下载了当前最新版本.

```sh
cd c:\opencv
git clone git@github.com:opencv/opencv.git sources
```

## 编译

打开 cmake-gui.exe, 然后将 source 目录下的 CMakeLists.txt 拖拽到打开的窗口中.
开始配置:

```yml
Where is the source code: C:/opencv/sources
Where to build the binaries: C:/opencv/opencv_build
```

然后点击 Configure. 检查一下标红的那些配置, 没有问题直接 Generate, 然后 Open Project.

在 Visual Studio 2017 里对 ALL_BUILD project 进行编译即可(Debug/Release).

## 注意事项

强烈建议将 Python 版本一起编译了, 可以下载安装最新的 Python3(目前是3.6.1), 并安装
[numpy](https://pypi.python.org/pypi/numpy).

强烈建议将例子一起编译了:

```sh
Check the box [X]BUILD_EXAMPLES
Check the box [X]INSTALL_PYTHON_EXAMPLES
```

也建议将文档一起编译了, 这个也可以省去, 在线文档已经足够方便. 如果需要编译文档, 那么,
[doxygen][2] 和 [graphviz][3] 是必备的. 然后:

```sh
Check the box [X]BUILD_DOCS
DOXYGEN_DOT_EXECUTABLE: D:/graphviz/bin/dot.exe
DOXYGEN_EXECUTABLE: D:/doxygen/doxygen.exe
```

## 安装

使用 OpenCV, 与使用其他库完全一致, 就是需要头文件(.h)和库文件(.lib). 在 Visual Studio 2017 中
对 INSTALL project 单独编译, 即执行了安装任务.

默认会生成在 `opencv_build/install` 下. 我们需要的主要是以下几个目录:

- `C:\opencv\opencv_build\install\include` --- 头文件
- `C:\opencv\opencv_build\install\x64\vc15\lib` --- 库文件
- `C:\opencv\opencv_build\install\x64\vc15\bin` --- 动态链接库

[0]: http://shop.oreilly.com/product/0636920044765.do
[1]: https://github.com/pezy/ReadingNotes/blob/master/learningOpenCV/01-Exercises.md
[2]: http://www.doxygen.org/
[3]: http://www.graphviz.org/
