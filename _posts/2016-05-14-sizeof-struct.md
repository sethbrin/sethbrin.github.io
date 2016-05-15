---
layout:     post
title:      "sizeof struct"
subtitle:   "关于sizeof(struct)大小"
date:       2016-05-14 23:20:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - c++
---

一般现在机器上各个常见的数据类型的大小为:

1. char 1byte
2. int(long) 4bytes
3. short 2bytes
4. double 8bytes
4. float 4bytes

# 规则
在没有#pragma pack的情况下:

> **1**. 数据成员对齐规则：结构体（struct）的数据成员，第一个数据成员放在offset为0的地方，之后的每个数据成员存储的起始位置要从该成员大小的整数倍开始（比如int在32位机子上为4字节，所以要从4的整数倍地址开始存储）。
>
> **2**. 结构体作为成员：如果一个结构体里同时包含结构体成员，则结构体成员要从其内部最大元素大小的整数倍地址开始存储（如struct a里有struct b,b里有char,int ,double等元素，那么b应该从8(即double类型的大小)的整数倍开始存储）。
>
> **3**. 结构体的总大小：即sizeof的结果。在按之前的对齐原则计算出来的大小的基础上，必须还得是其内部最大成员的整数倍，不足的要补齐（如struct里最大为double，现在计算得到的已经是11，则总大小为16）。

简单来说这几条规则，首先确定每个成员的首地址，对于double成员，其首地址是8的倍数，确定完所有的成员大小后，
对结构体整体有一个要求，整体大小必须是最大的成员大小的倍数，比如某个struct中最大成员大小为8,则整个大小必须是8的倍数，不足补齐。

{% highlight cpp %}
#include <iostream>

using namespace std;

#pragma pack(8)

struct One {
  double d;
  char c;
  int i;
}; // 8 + 1 + 3(padding) + 4 = 12 + 4 = 16

struct Two {
  char c;
  double d;
  int i;
}; // 1 + 7(padding) + 8 + 4 = 20 + 4 = 24

struct Three {
  struct One one;
  int c;
  char ch;
}; // 12 + 4 + 1 + 7(padding) = 24

struct Four {
  int c;
  struct One one; //12
  char ch;
}; // 4 + 4(padding) + 16 + 1 = 25 + 7 = 32

struct Five {
  short c;
  union {
    int a;
    short b;
  } u;
  double d;
}; // 2 + 2(padding) + 4 + 8=16

struct Six {
  short s;
  union {
    char c;
    short s;
  } u;
  double d;
}; // 2 + 2 + 4(padding) + 8 =16



int main()
{
  // 16:24:24:32

  cout << sizeof(struct One) << ":" << sizeof(struct Two) << ":" << sizeof(struct Three) << ":" << sizeof(struct Four)<< endl;

  // 16

  cout << sizeof(struct Five) << endl;
}
{% endhighlight %}

对于struct Four，成员c起始为0,大小为4, 成员one,按照其中最大的成员double 8位对齐，其大小为12,
然后成员ch是1位对齐，所以大小为4+4(补齐)+16+1=25，而整体大小需要是最大成员的整数倍，所以总大小为32.
