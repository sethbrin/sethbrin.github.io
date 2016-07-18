---
layout:     post
title:      "Clion use makefile"
subtitle:   " \"Clion 中使用自定义makefile\""
date:       2016-07-17 00:00:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - ide
    - c++
---

Clion中使用自定义Makefile
-------------------------

Clion默认使用CMake作为编译工具，一直说要集成其他工具，但是到现在
都没有，所以我们有时使用Clion来调试现有项目，但现有项目是基于自定义Makefile的，
这是就得配置CmakeList.txt来构造一个虚拟的target，调用项目的makefile。

{% highlight cpp %}

cmake_minimum_required(VERSION 3.4)
project(pro1)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99 -nostdinc")
SET(CMAKE_C_COMPILER gcc)
SET(CMAKE_CXX_COMPILER g++)

include_directories("/usr/include"
"/home/user/otherlib/include"
)

add_executable("pro1" main.cpp)

add_custom_target(project COMMAND make -C ${project_SOURCE_DIR}
CLION_EXE_DIR=${PROJECT_BINARY_DIR} && cp ${project_SOURCE_DIR}/project ./
)

{% endhighlight %}

注意其中定义pro1是一个伪造的目标，main.cpp也是一个随便定义的文件，一定要
加上这个，否则external library不自动导入。

add_custom_target这里加入你自己的项目
