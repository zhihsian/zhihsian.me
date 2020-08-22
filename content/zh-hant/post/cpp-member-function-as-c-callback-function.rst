---
title: "C++ 成員函數作 C 回調函數（模板）"
tags: ["C++"]
date: 2020-08-19T10:44:51+08:00
---

最近在用 C++ 寫一個網絡工具，使用了 libuv，回調的時候不能直接用成員函數太難受了，於是想辦法簡化寫法。

前提條件當然是幾乎所有 C 的回調函數都支持的透傳的自定義指針。

#. lambda
    lambda 是能夠最快想到的方式，但因爲得用 C 兼容的函數，所以不可以捕獲。這種方式無非就是把原本的靜態函數寫到行內了。

#. 宏
    第二想到的便是通過宏來生成 lambda 了，然而仔細想想，行不通，因爲參數、返回值並不確定，寫起來太醜陋。

#. 模板
    最後能用的就是模板了。

    先定義好要得到的東西：

    #. 成員函數所屬的類 :code:`C`，用來將指針轉爲對象指針
    #. 成員函數指針 :code:`F`
    #. 函數返回值 :code:`R`
    #. 函數第一個參數 :code:`Handle`，用來獲取指針
    #. 函數的其他參數 :code:`Args`

    開寫，過程不說了（

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

