---
layout:     post
title:      "atomic mutex"
subtitle:   "atomic和mutex对比"
date:       2016-09-06 20:01:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - cpp
    - concurent
---

# atomic和mutex的对比
代码如下:
{% highlight cpp %}
// g++ atomic_mutex.cpp -std=c++11 -lpthread

#include <atomic>

#include <future>

#include <thread>

#include <iostream>

#include <mutex>

#include "../util/util.h"

#define N 1000000

enum {
  ATOMIC,
  MUTEX,
};

std::atomic<long long> atomic_value(0);
void atomic_test() {
  for (int i = 0; i < N; i++) {
    atomic_value++;
  }
}

static int mutex_value = 0;
std::mutex mutex;
void mutex_test() {
  for (int i = 0; i < N; i++) {
    std::lock_guard<std::mutex> guard(mutex);
    mutex_value++;
  }
}

void benchmark(std::function<void ()> fun, int type) {
  double tt = 0;
  for (int i = 0; i < 10; i++) {
    double t = ::cputime();
    auto f = std::async(std::launch::async, fun);
    fun();
    f.wait();
    tt += (::cputime() - t);
  }
  std::string name;
  int val;
  switch(type) {
  case ATOMIC:
    name = "atomic";
    val = atomic_value;
    break;
  case MUTEX:
    name = "mutex";
    val = mutex_value;
    break;
  }
  std::cout << name << " ans:" << val << "  time:" << tt/10 << std::endl;
}

int main() {
  benchmark(atomic_test, ATOMIC);
  benchmark(mutex_test, MUTEX);
}

// atomic ans:20000000  time:0.120518

// mutex ans:20000000  time:0.589251
{% endhighlight %}
