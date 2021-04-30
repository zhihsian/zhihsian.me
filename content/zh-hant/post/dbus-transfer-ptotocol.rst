---
title: "D-Bus 通信協議"
tags: ["D-Bus"]
date: 2020-07-10T15:40:21+08:00
---

注：

#. 此文由 `D-Bus Specification <https://dbus.freedesktop.org/doc/dbus-specification.html>`_ 整理而來
#. 不贅述 D-Bus 的數據類型
#. 數據結構以類似於 C++ struct 僞代碼編寫

連接方式
********

Unix Domain Sockets
===================

\*nix 系統上的默認連接方式。

在一些分發版上，默認的地址爲：

* System Bus: :code:`/run/dbus/system_bus_socket`
* Session Bus: :code:`/run/user/[UID]/bus`

TCP Sockets
===========
通過 TCP 協議來傳輸數據

SystemD
=======

僅 Linux 系統可使用 SystemD，實際上 :code:`dbus-daemon` 使用 SystemD 的 socket activation 來創建基於 UDS 或 TCP 的服務端監聽，客戶端依然使用 UDS 或 TCP 來連接 D-Bus 服務。

實現方式：向 SystemD 提供 `*.socket` 和 `*.service` 配置文件。其中， `.socket` 中配置監聽地址， `.service` 中配置服務的啓動方式、依賴等。

有了 socket activation，服務甚至無需立即啓動，當有客戶端連接監聽的地址時，SystemD 負責開啓服務，然後將監聽的 socket 交給服務端。

launchd
=======
MacOS 上使用的服務管理系統，同樣使用 socket activation。

消息格式
********

D-Bus 消息包含消息頭和消息體。

消息頭的長度必須是 8 的倍數，若內容長度不是 8 的倍數，則必須以不多於 7 個字節的 0 進行填充。

消息體無長度必須是 8 的倍數的限制。

整個消息的長度必須是 2 到 2^27 字節（128 MiB），實現不允許發送或接受超過此大小的消息。

消息頭格式
==========

.. code-block:: c++

    struct message_header {
        byte_t   endianness_flag;   // 字節序，'l'（0x42）表示小端序，'B'（0x6C）表示大端序
        byte_t   message_type;      // 消息類型，未知類型必須被忽略，具體值在下面給出
        byte_t   flags;             // 標誌，按位每位的解釋下面表格給出
        byte_t   major;             // 協議主版本號，當前是 1
        uint32_t body_length;       // 消息體字節長度
        uint32_t serial;            // 消息的序列號，由發送方用作 cookie，以標識與此請求相對應的答覆，不允許爲 0
        array<field_item> fields;   // 零個或多個字段的數組，由消息類型決定哪些字段是必需的
    };

    struct field_item {
        byte_t  code;   // 字段
        variant value;  // 字段值
    };

array 封送格式見 `數組 <#數組>`_

variant 封送格式見 `variant <#variant>`_

類型
----

+-------------+--+-------------+
|名稱         |值|說明         |
+=============+==+=============+
|INVALID      |0 |無效的類型   |
+-------------+--+-------------+
|METHOD_CALL  |1 |方法調用     |
+-------------+--+-------------+
|METHOD_RETURN|2 |方法返回     |
+-------------+--+-------------+
|ERROR        |3 |錯誤返回     |
+-------------+--+-------------+

標誌
----

+-------------------------------+-------+------------------------------------------------------------------------+
|名稱                           |值     |說明                                                                    |
+===============================+=======+========================================================================+
|NO_REPLY_EXPECTED              |1 << 0 |此消息不期望方法返回或錯誤返回                                          |
|                               |       |                                                                        |
|                               |       |METHOD_CALL 是唯一可以期望方法返回或錯誤返回的消息類型，所以此標誌對其它|
|                               |       |消息類型是無意義的                                                      |
+-------------------------------+-------+------------------------------------------------------------------------+
|NO_AUTO_START                  |1 << 1 |若是目的服務沒啓動，D-Bus 不應該啓動它                                  |
+-------------------------------+-------+------------------------------------------------------------------------+
|ALLOW_INTERACTIVE_AUTHORIZATION|1 << 2 |可以在方法調用消息上設置此標誌，以通知接收方調用者已準備好等待交互式授權|
|                               |       |，這可能需要花費大量時間才能完成。例如：如果設置了此標誌，則可以通過    |
|                               |       |Polkit 或類似框架向用戶查詢密碼或確認。                                 |
|                               |       |                                                                        |
|                               |       |該標誌僅對方法調用消息有效，否則將被忽略。客戶端不應該默認設置此標誌，僅|
|                               |       |當非特權代碼調用特權更大的方法調用並且部署了允許進行交互式授權的授權框架|
|                               |       |時，此標誌纔有用。如果沒有部署這樣的框架，它將沒有效果。如果已設置，則調|
|                               |       |用方還應在方法調用上設置適當的長時間超時，以確保用戶交互可以完成。      |
|                               |       |                                                                        |
|                               |       |如果未在方法調用上設置此標誌，並且服務確定在沒有交互式授權的情況下不允許|
|                               |       |請求的操作，但是在成功進行交互式授權後可以允許該操作，則它可能返回      |
|                               |       |org.freedesktop.DBus.Error.InteractiveAuthorizationRequired 錯誤。      |
|                               |       |                                                                        |
|                               |       |缺少此標誌並不能保證不會應用交互式授權，因爲在此標誌之前的現有服務可能已|
|                               |       |經使用了交互式授權。但是，將使用交互式授權的現有 D-Bus API              |
|                               |       |應該記錄該調用可能比平時花費更長的時間，並且新的 D-Bus API              |
|                               |       |應該在沒有此標誌的情況下避免交互式授權。                                |
+-------------------------------+-------+------------------------------------------------------------------------+

字段
----

+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|名稱           |字段代碼   |字段值類型 |必傳               |控制       |說明                                                  |
+===============+===========+===========+===================+===========+======================================================+
|INVALID        |0          |N/A        |不允許             |           |                                                      |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|PATH           |1          |objectPath | | METHOD_CALL     |發送者     |要發送呼叫的對象或發出信號的對象。特殊路徑            |
|               |           |           | | SIGNAL          |           |/org/freedesktop/DBus/Local 是保留路徑；實現不應使用此|
|               |           |           |                   |           |路徑發送消息，並且總線守護程序的參考實現將斷開任何嘗試|
|               |           |           |                   |           |這樣做的應用程序的連接。                              |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|INTERFACE      |2          |string     |SIGNAL             |發送者     |調用方法調用或發出信號的接口。對於方法調用是可選的，對|
|               |           |           |                   |           |於信號是必需的。 特殊接口 org.freedesktop.DBus.Local  |
|               |           |           |                   |           |是保留接口；實現不應使用此接口發送消息，並且總線守護程|
|               |           |           |                   |           |序的參考實現將斷開任何嘗試這樣做的應用程序的連接。    |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|MEMBER         |3          |string     | | METHOD_CALL     |發送者     |成員名，比如方法名、信號名。                          |
|               |           |           | | SIGNAL          |           |                                                      |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|ERROR_NAME     |4          |string     |ERROR              |           |發生的錯誤的名稱，用於錯誤。                          |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|REPLY_SERIAL   |5          |uint32     | | METHOD_RETURN   |發送者     |用於標明返回數據所答覆的調用的序列號（也就是調用時，消|
|               |           |           | | ERROR           |           |息頭中的 serial），用於告知調用者這是哪個調用的返回。 |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|DESTINATION    |6          |string     |可選               |發送者     |此消息打算用於的連接的名稱。該字段通常僅與消息總線結合|
|               |           |           |                   |           |使用纔有意義，但其他服務器可能會爲其定義自己的含義。  |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|SENDER         |7          |string     |可選               |消息總線   |發送鏈接的唯一名稱，該字段由消息總線控制，所以它的值與|
|               |           |           |                   |           |消息總線本身一樣可靠且值得信賴。                      |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|SIGNATURE      |8          |signature  |可選               |發送者     |發送的數據的簽名，如果沒有此字段，則假定它是空的，此時|
|               |           |           |                   |           |消息體也必須是空的。                                  |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+
|UNIX_FDS       |9          |uint32     |可選               |發送者     |消息附帶的 Unix                                       |
|               |           |           |                   |           |文件描述符的數量。 如果省略，則假定該消息中沒有 Unix  |
|               |           |           |                   |           |文件描述符。實際文件描述符需要通過特定於平臺的機制進行|
|               |           |           |                   |           |帶外傳輸。它們必須作爲消息本身的一部分同時發送。在消息|
|               |           |           |                   |           |本身的第一個字節被傳輸之前或消息本身的最後一個字節之後|
|               |           |           |                   |           |，可能不會發送它們。 該頭字段由消息發送者控制。       |
+---------------+-----------+-----------+-------------------+-----------+------------------------------------------------------+

消息體格式
==========

可以看到，消息頭已經描述了消息的協議版本、daemon 收到消息後的操作、來源、目的，剩下的消息體就只需要放數據了，比如方法調用的參數、方法的返回值、方法的錯誤返回、信號的值。

消息體的結構很簡單，就是將數據連接封送到一起。

數據封送
========

對齊
----

字節塊中的每個值都「自然」對齊，例如 4 字節值與 4 字節邊界對齊、8 字節值與 8 字節邊界對齊。對齊的邊界是全局計算的，即相對於整個消息的第一個字節計算。要對齊一個值，可能需要在該值之前添加對齊填充。

字符串、容器，由於數據大小並不固定，要做到邊界對齊，不僅需要在數據之前對齊填充，還可能需要在後面填充，他們的填充大小規則見下文。

作爲自然對齊的一個例外，struct 和 dictEntry 的值始終與 8 字節邊界對齊，而不考慮其值的類型。

對齊填充必須遵循以下標準：

* 必須是以正確對齊爲前提的最小填充
* 必須由「0」組成

以下以 nature_align(T) 表示類型 T，或者值 T 的類型的自然對齊大小。

字符串類型（string、objectPath、signature）
-------------------------------------------

.. code-block:: c++

    struct string {
        uint32_t length;
        byte_t   data[length];
        byte_t   end_zero[1];
    };

    typedef string objectPath;

    struct signature {
        uint8_t length;
        byte_t  data[length];
        byte_t  end_zero[1];
    };

字符串類型的結構包含 4 部分：

1. 用來描述字符串長度的無符號整數，其中，string、objectPath 以 uint32 來描述，signature 以 uint8 來描述
2. 緊接着的是字符串內容
3. 後置 0（不被計入字符串長度）
4. 填充 0

字符串結構整體需要以 :code:`nature_align(length)` 邊界來對齊，即 :code:`nature_align(string) == nature_align(string.length)` ，若是不足，則需要以小於 :code:`nature_align(length)` 個 0 來填充

容器
----

數組
^^^^

.. code-block:: c++

    struct array<T> {
        uint32_t length;            // 數組內容的字節大小
        byte_t   items_padding_zero[(offset(length) + sizeof(length) + (nature_align(T) - 1)) & ~(nature_align(T) - 1)];
        T        items[];
    };

數組整體同樣需要以 :code:`nature_align(length)` 字符邊界來對齊，即 :code:`nature_align(array) == nature_align(array.length)`

數組中的每一項，需要以 :code:`nature_align(T)` 字符邊界來對齊，所以 length 與 items[] 之間，可能需要不小於 :code:`nature_align(T)` 個填充 0，而每一項之間，由於類型相同，所以緊密連接時不需要再有額外的填充來使之對齊

struct、dictEntries
^^^^^^^^^^^^^^^^^^^

與數組類似，不同的是，由於不同的項可能是不同的類型，所以他們之間會有填充，使每一項能夠有與自己的類型相關的對齊。

由於在消息頭中的 fields 裏的 SIGNATURE 項，已經標明瞭數據類型，所以 struct 和 dictEntry 不需要再聲明容器中值的類型了。

variant
^^^^^^^

.. code-block:: c++

    struct variant {
        signature sign;     // variant 包含數據的數據類型
        byte_t    value_padding_zero[(offset(sign) + sizeof(sign) + (nature_align(T) - 1)) & ~(nature_align(T) - 1)];
        T         value;
    };

variant 整體以 :code:`nature_align(sign)` 字符邊界來對齊，即 :code:`nature_align(varian) == nature_align(variant.sign)`

而 variant 的值，以 :code:`nature_align(T)` 字符邊界來對齊，所以 sign 和 value 之間，可能需要不小於 :code:`nature_align(T)` 個填充 0

例子
****

.. code-block:: c++

    struct message_header {
        byte_t   endianness_flag;                       // 6c               'l'，小端序
        byte_t   message_type;                          // 01               0，METHOD_CALL
        byte_t   flags;                                 // 00               0，無 flag
        byte_t   major;                                 // 01               1，版本號 1
        uint32_t body_length;                           // 32 00 00 00      消息體長 0x32 字節
        uint32_t serial;                                // 58 02 00 00      消息序列號 0x0258
        struct array<field_item> {
            uint32_t length;                            // 76 00 00 00      數組內容字節長度 0x76 字節
            byte_t   items_padding_zero[0];             //                  這裏剛好是 8 的倍數，所以無需 padding
            struct field_item {
                byte_t  code;                           // 08               SIGNATURE
                struct variant {
                    struct signature {
                        uint8_t length;                 // 01               字符串長度 1
                        byte_t  data[length];           // 67               'g'
                        byte_t  end_zero[1];            // 00
                    } sign;
                    byte_t    value_padding_zero[0];    //                  T 是 signature，以 1 對齊，所以無需填充
                    struct signature {
                        uint8_t length;                 // 02               字符串長度 2
                        byte_t  data[length];           // 73 73            "ss"
                        byte_t  end_zero[1];            // 00
                    } value;
                } value;
            } item0;
            struct filed_item {
                byte_t  code;                           // 01               PATH
                struct variant {
                    struct signature {
                        uint8_t length;                 // 01               字符串長度 1
                        byte_t  data[length];           // 6f               'o'
                        byte_t  end_zero[1];            // 00
                    } sign;
                    byte_t    value_padding_zero[0];    //                  objectPath 以 4 字節邊界對齊，這裏剛好，無需填充
                    struct string {
                        uint32_t length;                // 1d 00 00 00      29
                        byte_t   data[length];          // 2f 63 6f 6d      "/com/deepin/daemon/SystemInfo"
                                                        // 2f 64 65 65
                                                        // 70 69 6e 2f
                                                        // 64 61 65 6d
                                                        // 6f 6e 2f 53
                                                        // 79 73 74 65
                                                        // 6d 49 6e 66
                                                        // 6f
                        byte_t   end_zero[1];           // 00
                        byte_t   end_padding[3]         // 00 00            字符串整體需要以 length 的類型的邊界對齊，所以需要填充倆 0
                    };
                } value;
            } item1;
            struct field_item {
                byte_t  code;                           // 03               MEMBER
                struct variant {
                    struct signature {
                        uint8_t length;                 // 01               字符串長度 1
                        byte_t  data[length];           // 73               's'
                        byte_t  end_zero[1];            // 00
                    } sign;
                    byte_t    value_padding_zero[0];    //                  string 以 4 字節邊界對齊，這裏剛好，無需填充
                    struct string {
                        uint32_t length;                // 03 00 00 00      長度 0x03
                        byte_t   data[length];          // 47 65 74         "Get"
                        byte_t   end_zero[1];           // 00
                    };
                } value;
            } item2;
                                                        // 00 00 00 00      D-Bus 的 struct 需要以 8 字節邊界對齊，所以這裏填充 4 字節的 0
            struct field_item {
                byte_t  code;                           // 02               INTERFACE
                struct variant {
                    struct signature {
                        uint8_t length;                 // 01               字符串長度 1
                        byte_t  data[length];           // 73               's'
                        byte_t  end_zero[1];            // 00
                    } sign;
                    byte_t    value_padding_zero[0];    //                  string 以 4 字節邊界對齊，這裏剛好，無需填充
                    struct string {
                        uint32_t length;                // 1f 00 00 00      31
                        byte_t   data[length];          // 6f 72 67 2e      "org.freedesktop.DBus.Properties"
                                                        // 66 72 65 65
                                                        // 64 65 73 6b
                                                        // 74 6f 70 2e
                                                        // 44 42 75 73
                                                        // 2e 50 72 6f
                                                        // 70 65 72 74
                                                        // 69 65 73 00
                                                        // 06 01 73
                        byte_t   end_zero[1];           // 00
                    };
                } value;
            } item3;
            struct field_item {
                byte_t  code;                           // 06               DESTINATION
                struct variant {
                    struct signature {
                        uint8_t length;                 // 01               字符串長度 1
                        byte_t  data[length];           // 73               's'
                        byte_t  end_zero[1];            // 00
                    } sign;
                    byte_t    value_padding_zero[0];    //                  string 以 4 字節邊界對齊，這裏剛好，無需填充
                    struct string {
                        uint32_t length;                // 05 00 00 00      長度 0x05
                        byte_t   data[length];          // 3a 31 2e 32      ":1.27"
                                                        // 37
                        byte_t   end_zero[1];           // 00
                                                        //                  到這裏 header 已經結束，所以雖然 string 末尾沒有到邊界，依然不需要再填充
                    };
                } value;
            } item4;
        } fields;
    };
    byte_t   padding[3];                                // 00 00            字符串整體需要以 length 的類型的邊界對齊，所以需要填充倆 0
    struct string {
        uint32_t length;                                // 1c 00 00 00      長度 28
        byte_t   data[length];                          // 63 6f 6d 2e      "com.deepin.daemon.SystemInfo"
                                                        // 64 65 65 70
                                                        // 69 6e 2e 64
                                                        // 61 65 6d 6f
                                                        // 6e 2e 53 79
                                                        // 73 74 65 6d
                                                        // 49 6e 66 6f
        byte_t   end_zero[1];                           // 00
        byte_t   end_padding[3]                         // 00 00 00         字符串整體需要以 length 的類型的邊界對齊，所以需要填充仨 0
    };
    struct string {
        uint32_t length;                                // 09 00 00 00      長度 9
        byte_t   data[length];                          // 63 6f 6d 2e      "Processor"
        byte_t   end_zero[1];                           // 00
    };
                                                        //                  已經結束，無需填充
