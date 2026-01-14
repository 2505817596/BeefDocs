+++
title = "核心库"
weight = 40
+++

## Beef 核心库概览

Beef 的基础标准库称为 "corlib"，提供基础工具类型与通用系统功能。

以下为 corlib 提供的部分集合类型。

|类别|类型|
|-----|------|
|列表|[System.Collections.List<T>](../doxygen/corlib/html/class_system_1_1_collections_1_1_list.html)|
|字典|[System.Collections.Dictionary<T>](https://github.com/beefytech/Beef/blob/master/BeefLibs/corlib/src/Collections/Dictionary.bf)|
|哈希集合|[System.Collections.HashSet<T>](../doxygen/corlib/html/class_system_1_1_collections_1_1_hash_set.html)|
|队列|[System.Collections.Queue<T>](../doxygen/corlib/html/class_system_1_1_collections_1_1_queue.html)|

以下为 corlib 提供的部分功能，完整列表请参见 API 文档。

|类别|类型|
|-----|------|
|字符串|[System.String](../doxygen/corlib/html/class_system_1_1_string.html)<br/>[System.StringView](../doxygen/corlib/html/struct_system_1_1_string_view.html)|
|数学|[System.Math](../doxygen/corlib/html/class_system_1_1_math.html)|
|随机|[System.Random](../doxygen/corlib/html/class_system_1_1_random.html)|
|错误处理|[System.Result<T>](../doxygen/corlib/html/struct_system_1_1_result.html)|
|文件与目录|[System.IO.File](../doxygen/corlib/html/class_system_1_1_i_o_1_1_file.html)<br/>[System.IO.FileStream](../doxygen/corlib/html/class_system_1_1_i_o_1_1_file_stream.html)<br/>[System.IO.Directory](../doxygen/corlib/html/class_system_1_1_i_o_1_1_directory.html)<br/>[System.IO.Path](../doxygen/corlib/html/class_system_1_1_i_o_1_1_path.html)|
|日期与时间|[System.DateTime](../doxygen/corlib/html/struct_system_1_1_date_time.html)|
|计时|[System.Diagnostics.Stopwatch](../doxygen/corlib/html/class_system_1_1_diagnostics_1_1_stopwatch.html)|
|套接字|[System.Net.Socket](../doxygen/corlib/html/class_system_1_1_net_1_1_socket.html)|
|线程|[System.Threading.Thread](../doxygen/corlib/html/class_system_1_1_threading_1_1_thread.html)<br/>[System.Threading.ThreadPool](../doxygen/corlib/html/class_system_1_1_threading_1_1_thread_pool.html)<br/>[System.Threading.Monitor](../doxygen/corlib/html/class_system_1_1_threading_1_1_monitor.html)<br/>[System.Threading.WaitEvent](../doxygen/corlib/html/class_system_1_1_threading_1_1_wait_event.html)|
|原子操作|[System.Threading.Interlocked](../doxygen/corlib/html/class_system_1_1_threading_1_1_interlocked.html)|
|控制台|[System.Console](../doxygen/corlib/html/class_system_1_1_console.html)|
|调试辅助|[System.Diagnostics.Debug](../doxygen/corlib/html/class_system_1_1_diagnostics_1_1_debug.html)|
|哈希|[System.Cryptography.MD5Hash](../doxygen/corlib/html/class_system_1_1_security_1_1_cryptography_1_1_m_d5.html)<br/>[System.Cryptography.SHA256](../doxygen/corlib/html/struct_system_1_1_security_1_1_cryptography_1_1_s_h_a256.html)|
|文本编码|[System.Text.Encoding](../doxygen/corlib/html/class_system_1_1_text_1_1_encoding.html)|
|多播委托|[System.Event<T>](../doxygen/corlib/html/struct_system_1_1_event.html)|
|FFI|[System.FFI.FFILIB](../doxygen/corlib/html/struct_system_1_1_f_f_i_1_1_f_f_i_l_i_b.html)<br/>[System.FFI.FFIType](../doxygen/corlib/html/struct_system_1_1_f_f_i_1_1_f_f_i_type.html)<br/>[System.FFI.FFICaller](../doxygen/corlib/html/struct_system_1_1_f_f_i_1_1_f_f_i_caller.html)|
|Windows API|[System.Windows](../doxygen/corlib/html/class_system_1_1_windows.html)|

## 多媒体库

Beef IDE 使用名为 Beefy2D 的窗口与多媒体库。该库仅供 IDE 与其他内部工具使用，不面向第三方应用，API 也可能在不保证向后兼容的情况下变更。因此第三方应用应使用其他第三方库。

第三方多媒体库的一个例子是 SDL2，其用法可在随附的 Beef 示例中看到。
