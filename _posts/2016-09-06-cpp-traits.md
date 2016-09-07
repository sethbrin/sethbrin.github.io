---
layout:     post
title:      "Cpp traits"
subtitle:   "C++的一些技巧"
date:       2016-09-06 15:01:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - cpp
---

## C++ Template中防止使用不完整类型
查看如下代码可以知道原因

{% highlight cpp %}
template<class T>
class A {
 private:
  int i;
};

int main()
{
  A<int> a;
  return 0;
} //编译无问题

//Code 2
class B;

template<class T>
class A {
 private:
  int i;
};

int main()
{
  A<B> a;
  return 0;
}
// 编译无问题,但B的类型不明确

//Code 3
class B;

template<class T>
class A {
 public:
  A() { enum{x=sizeof(B)}; }
 private:
  int i;
};

int main()
{
  A<B> a;
  return 0;
}
>>编译出错
>>tt.cpp: In constructor ‘A<T>::A()’:
>>tt.cpp:6: error: invalid application of ‘sizeof’ to incomplete type ‘B’
{% endhighlight %}
看了上面的描述大家应该知道这个的作用了，所以这种技巧常用在一些模版类，如scope_ptr中
{% highlight cpp %}
void reset(C* p = NULL) {
  if (p != ptr_) {
  enum { type_must_be_complete = sizeof(C) };
  delete ptr_;
  ptr_ = p;
}

{% endhighlight %}

## operator void*()
如下代码:
{% highlight cpp %}
class Foo {
 public:
  operator void* () {
    return ptr;
  }

 private:
  void *ptr;
};
{% endhighlight %}
上面operator void*() 是类型转换函数，用在

<code>
    Foo foo;
    void* ptr = foo;  // The compiler will call `operator void*` here
</code>

而对于operator() 是一个函数调用符

<code>
    Foo foo;
    void* ptr = foo();  // The compiler calls `operator()` here
</code>
