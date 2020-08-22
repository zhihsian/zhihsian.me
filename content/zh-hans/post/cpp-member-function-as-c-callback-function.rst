---
title: "C++ 成员函数作 C 回调函数（模板）"
tags: ["C++"]
date: 2020-08-19T10:44:51+08:00
---

最近在用 C++ 写一个网络工具，使用了 libuv，回调的时候不能直接用成员函数太难受了，于是想办法简化写法。

前提条件当然是几乎所有 C 的回调函数都支持的透传的自定义指针。

#. lambda
    lambda 是能够最快想到的方式，但因为得用 C 兼容的函数，所以不可以捕获。这种方式无非就是把原本的静态函数写到行内了。

#. 宏
    第二想到的便是通过宏来生成 lambda 了，然而仔细想想，行不通，因为参数、返回值并不确定，写起来太丑陋。

#. 模板
    最后能用的就是模板了。

    先定义好要得到的东西：

    #. 成员函数所属的类 :code:`C`，用来将指针转为对象指针
    #. 成员函数指针 :code:`F`
    #. 函数返回值 :code:`R`
    #. 函数第一个参数 :code:`Handle`，用来获取指针
    #. 函数的其他参数 :code:`Args`

    开写，过程不说了（

    .. code-block:: c++

        template <auto F>
        struct CallbackWrapper;
        template <typename C,
                  typename R,
                  typename Handle,
                  typename... Args,
                  R (C::*F)(Handle, Args...)>
        struct CallbackWrapper<F> {
            static R func(Handle handle, Args... args) {
                auto *p = reinterpret_cast<C *>(handle->data);
                return (p->*F)(handle, args...);
            }
        };

