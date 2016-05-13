---
layout:     post
title:      "Why name hiding occur in derived class"
subtitle:   " \"C++函数隐藏的设计原因\""
date:       2016-05-12 10:30:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - c++
    - name hiding
---

基本概念
====
覆盖(override)，隐藏(hide)和重载(overload)这是c++里很重要也很基本的概念，
其中隐藏和覆盖是基于继承体系来讨论的，而重载的概念很简单，只是针对参数不同的同名函数.
下面举几个例子来说明他们之间的异同点：

## 重载

> 必须在一个域中,函数名称相同但是函数参数不同,重载的作用就是同一个函数有不同的行为,
> 因此不是在一个域中的函数是无法构成重载的,这个是重载的重要特征

{% highlight cpp %}
class A {
 public:
  void fun(int a){
    printf("int\n");
  }
  void fun(float a){
    printf("int\n");
  }
};

int main() {
  A a;
  a.fun(2); // int

  a.fun(2.0f); // float
  return 0;
}
{% endhighlight %}

## 覆盖

> 覆盖也就是我们常说的虚函数，它有两个必要条件：
> 1. 有virtual关键字，在基类中函数声明的时候加上就可以了
> 2. 基类中的函数和派生类中的函数要一模一样，什么叫一模一样，函数名，参数，返回类型三个条件

{% highlight cpp %}
class A {
 public:
  virtual void fun(int a) {
    printf("A\n");
  }
};

class B : A {
 public:
  virtual void fun(int a) {
    printf("B\n");
  }
};

int main() {
  A a;
  B b;

  a.fun(1); // A

  b.fun(1); // B
  return 0;
}
{% endhighlight %}

## 隐藏

> “隐藏”是指派生类的函数屏蔽了与其同名的基类函数, 规则如下：
> 1. 如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）。
> 2. 如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）。

{% highlight cpp %}
class A {
 public:
  void fun1(int a) {
    printf("A: fun1(int)\n");
  }
  virtual void fun2(int a) {
    printf("A: fun2(int)\n");
  }

  void fun3(int a) {
    printf("A: fun3(int)\n");
  }
};

class B : A {
 public:
  void fun1(float a) {
    printf("B: fun1(float)\n");
  }
  virtual void fun2(float a) {
    printf("B: fun2(float)\n");
  }

  void fun3(int a) {
    printf("B: fun3(int)\n");
  }
};
int main() {
  B b;

  b.fun1(1.0f); // B: fun1(int)

  b.fun2(1.0f); // B: fun2(int)

  b.fun3(1); // B: fun3(int)

  return 0;
}
{% endhighlight %}
[详细代码](https://github.com/sethbrin/recipes/blob/master/basic/override_overload_hiding.cpp)

## 为什么要定义隐藏[[3]](#ref3)？

> Judging by the wording of your question (you used the word "hide"), you already know what is going on here. The phenomenon is called "name hiding". For some reason, every time someone asks a question about why name hiding happens, people who respond either say that this called "name hiding" and explain how it works (which you probably already know), or explain how to override it (which you never asked about), but nobody seems to care to address the actual "why" question.

> The decision, the rationale behind the name hiding, i.e. why it actually was designed into C++, is to avoid certain counterintuitive, unforeseen and potentially dangerous behavior that might take place if the inherited set of overloaded functions were allowed to mix with the current set of overloads in the given class. You probably know that in C++ overload resolution works by choosing the best function from the set of candidates. This is done by matching the types of arguments to the types of parameters. The matching rules could be complicated at times, and often lead to results that might be perceived as illogical by an unprepared user. Adding new functions to a set of previously existing ones might result in a rather drastic shift in overload resolution results.

> For example, let's say the base class B has a member function foo that takes a parameter of type void *, and all calls to foo(NULL) are resolved to B::foo(void *). Let's say there's no name hiding and this B::foo(void *) is visible in many different classes descending from B. However, let's say in some [indirect, remote] descendant D of class B a function foo(int) is defined. Now, without name hiding D has both foo(void *) and foo(int) visible and participating in overload resolution. Which function will the calls to foo(NULL) resolve to, if made through an object of type D? They will resolve to D::foo(int), since int is a better match for integral zero (i.e. NULL) than any pointer type. So, throughout the hierarchy calls to foo(NULL) resolve to one function, while in D (and under) they suddenly resolve to another.

> This behavior was deemed undesirable when the language was designed. As a better approach, it was decided to follow the "name hiding" specification, meaning that each class starts with a "clean sheet" with respect to each method name it declares. In order to override this behavior, an explicit action is required from the user: originally a redeclaration of inherited method(s) (currently deprecated), now an explicit use of using-declaration.

> As you correctly observed in your original post (I'm referring to the "Not polymorphic" remark), this behavior might be seen as a violation of IS-A relationsip between the classes. This is true, but apparently back then it was decided that in the end name hiding would prove to be a lesser evil.

隐藏的主要原因主要是为了防止某些违反直觉的，无法预见的，具有潜在危险的操作发生。C++重载决议为了寻找最合适的函数，主要通过寻找最佳的参数匹配来执行的，添加新的备选函数可能导致在决议过程中很诡异的现象，
下面举一个例子来说明：
假设B含有函数foo(void *), 所有调用foo(NULL)会被解析成B::foo(void *),假设没有隐藏，B::foo(void *)
对于在B的继承链下游的Class都可见，对于其中某个类D（可能不直接继承B，B->C->D），其中D定义了函数foo(int),
这时调用foo(NULL)时就无法正常解析都，对于继承链比较长的情况，就很难知道某个函数是否被重载，
所以定义了隐藏规则，假设没有隐藏规则，对于上述类型的决议，显然重载是不可取的，因为继承链太长，
定义覆盖这个规则可能会导致无法预见的问题，所以覆盖是不能跨作用域的，这也是上面提到的，
而对于覆盖，必须函数申明一致，基于这些原因，定义一种新的函数决议规则--隐藏。

参考文档
------------------
1. <span id="ref1"></span>[谈谈C++继承中的重载，覆盖和隐藏][1]
2. <span id="ref2"></span>[Bjarne Stroustrup's C++ Style and Technique FAQ][2]
3. <span id="ref3"></span>[Why does an overridden function in the derived class hide other overloads of the base class?][3]

[1]: http://www.cnblogs.com/xubin0523/archive/2012/05/31/2528968.html
[2]: http://www.stroustrup.com/bs_faq2.html#overloadderived
[3]: http://stackoverflow.com/questions/1628768/why-does-an-overridden-function-in-the-derived-class-hide-other-overloads-of-the
