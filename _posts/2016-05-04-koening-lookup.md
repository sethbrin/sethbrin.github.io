---
layout:     post
title:      "Koening lookup"
subtitle:   "C++名称查找规则之-Koening lookup"
date:       2016-05-04 10:58:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - c++
---

C++名称查找规则
-----------------

* OL(Ordinal lookup): 顺序查找 [[1]](#ref1) 

  从函数调用所处的域开始时(如果函数调用处于一个成员函数中，初始域就是类域，如果处于自由函数中，初始域就是名字空间域或者
  全局域)，依次由内到外到各个域进行名字查找，如果某个域找到该名字的函数，就停止查找，将在该域找到所有重载函数进行重载决议，
  如果没有合适的候选者或者有多个合适的候选者导致歧义，编译器就报错，如果编译器一直没有找到也同样报错。

  > 如果通过一个类实例调用类的成员函数，则从该实例对应的类域开始，到该类的基类(如果有的话)，然后再是实例调用类的成员函数
  > 所在的类域开始.

{% highlight cpp %}
namespace N1
{
  namespace N2
  {
    class A
    {
      public:
        void fun1()
        {
          fun();
        }
    }
  }
}
{% endhighlight %}

  > 调用fun函数时，查找的顺序是A->N2->N1->全局域
  >
  > OL是名字查找的主要规则，只是OL应用的某些阶段KL也起作用，并将其作用附加在OL上。

* Koening Lookup: 又称ADL(argument-dependment lookup),参数相关查找，是指编译器对无限定域的函数调用进行名字查找,除了当前名字空间
  域以外，也会把函数参数类型所处的名称空间加入查找范围。[[2]](#ref2)
    1. f(x, y, z) // unqualified N::f(x, y, z) // qualified
    2. Herb 提供的解释(Exception C++, Item 31)
      > Koenig Lookup(simplified): If you supply a function argument of class type (here x, of type A::X), then to look up the correct function name the compiler considers matching names in the namespace (here A) containing the argument's type.

{% highlight cpp %}

namespace N1 {
  struct S {};
  template<int X> void f(S) {
    std::cout << "N1" << std::endl;
  };
}
namespace N2 {
  template<class T> void f(T t) {
    std::cout << "N2" << std::endl;
  }
}

template<int X> void f(N1::S) {
  std::cout << "N3" << std::endl;
}
void g(N1::S s) {
  f<3>(s);      // Syntax error (unqualified lookup finds no f)

  N1::f<3>(s);  // OK, qualified lookup finds the template 'f'

  N2::f<3>(s);  // Error: N2::f does not take a non-type parameter

                //        N1::f is not looked up because ADL only works

                //              with unqualified names

  f<3>(s); // OK: Unqualified lookup now finds N2::f

           //     then ADL kicks in because this name is unqualified

           //     and finds N1::f
}

{% endhighlight %}

详细可参考Exceptional c++ Item 31

参考文档
------------------
1. <span id="ref1"></span>[编译器名字查找规则之adl和ordinal-lookup比较][1]
2. <span id="ref2"></span>[Argument-dependent lookup][2]

[1]: http://www.zyh1690.org/编译器名字查找规则之adl和ordinal-lookup比较/
[2]: http://en.cppreference.com/w/cpp/language/adl
