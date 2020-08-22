---
title: "VSCode 使用 clangd 來實現代碼提示、補全"
tags: ["C++", "VSCode"]
date: 2020-08-21T22:55:24+08:00
---

前天寫了 `C++ 成員函數作 C 回調函數（模板） <{{< ref "cpp-member-function-as-c-callback-function.rst" >}}>`_ 後，VSCode 的 C/C++ 插件一直報 :code:`incomplete type is not allowed`，所以決定換到 clangd 試試，結果發現效果超出預期～

配置 CMake
**********
與 C/C++ 插件一樣，clang 插件需要使用 compile_commands.json 文件，但它並不支持 configuration provider，所以需要手動配置 compile_commands.json 的路徑。

然而，我的 CMake 是生成路徑是通過構建工具和構建類型來生成的，而 clangd 插件並不支持使用這些變量，所以得讓 CMake 插件複製下 compile_commands.json：

.. code-block:: json

    {
        "cmake.copyCompileCommands": "${workspaceFolder}/build/compile_commands.json",
    }

配置 clangd
***********
因爲我的系統默認安裝的 clangd-9 還有些問題：調用函數的實參有使用到模板時，無法跳轉到函數定義。所以裝了 clangd-10，需要手動指定下 clangd-10 的路徑。

.. code-block:: json

    {
        "clangd.path": "/usr/bin/clangd-10",
        "clangd.arguments": [
            "--compile-commands-dir=${workspaceFolder}/build",
            "--background-index",
            "--clang-tidy",
            "--log=verbose",
            "--pretty",
        ],
    }

關閉 C/C++ 插件衝突的功能
*************************
因爲還需要 Debug，所以不能完全禁用掉 C/C++ 插件。不過，像智能提示、自動補全、錯誤提示就都可以禁掉啦。

.. code-block:: json

    {
        "C_Cpp.intelliSenseEngine": "Disabled",
        "C_Cpp.autocomplete": "Disabled",
        "C_Cpp.errorSquiggles": "Disabled",
    }

