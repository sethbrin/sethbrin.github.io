---
layout:     post
title:      "bind and function"
subtitle:   "\"c++11 bind和function\""
date:       2016-09-6 23:00:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - cpp
---

## bind的用法
下面是bind的函数申明：

{% highlight cpp %}

// boost

template<class F, class A1, class A2, class A3>
_bi::bind_t<_bi::unspecified, F, typename _bi::list_av_3<A1, A2, A3>::type>
BOOST_BIND(F f, A1 a1, A2 a2, A3 a3)
{
  typedef typename _bi::list_av_3<A1, A2, A3>::type list_type;
  return _bi::bind_t<_bi::unspecified, F, list_type>(f, list_type(a1, a2, a3));
}

// c11

template<typename _Result, typename _Func, typename... _BoundArgs>
inline
typename _Bindres_helper<_Result, _Func, _BoundArgs...>::type
bind(_Func&& __f, _BoundArgs&&... __args)
{
  typedef _Bindres_helper<_Result, _Func, _BoundArgs...> __helper_type;
  typedef typename __helper_type::__maybe_type __maybe_type;
  typedef typename __helper_type::type __result_type;
  return __result_type(__maybe_type::__do_wrap(std::forward<_Func>(__f)),
      std::forward<_BoundArgs>(__args)...);
}

{% endhighlight %}

上面可以看出，bind默认是值传递的.如果要使用引用传递用std::ref或者std::cref.

## function的用法
函数对象很好理解，就是把函数指针封装到对象中，常和bind搭配使用，如:
{% highlight cpp %}
int fun(int a, int b) {
  return a + b;
}

std::function<int (int, int)> f = fun;
f(1, 2);
std::function<int (int)> f = std::bind(fun, 2, std::placeholders::_1);
f(3);
{% endhighlight %}
