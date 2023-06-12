# sese-core

> 跨平台类库

## 开始

本库依赖如下

| 名称            | 地址                                  | 描述           |
|---------------|-------------------------------------|--------------|
| googletest    | github.com/google/googletest        | 用于单元测试       |
| openssl       | github.com/openssl/openssl          | SSL 支持       |
| hpack-rfc7541 | github.com/jnferguson/hpack-rfc7541 | HUFFMAN 编码解码 |
| zlib          | zlib.net                            | 压缩算法         |
| SimpleUuid    | github.com/libsese/SimpleUuid       | 雪花算法实现       |
| SString       | github.com/libsese/SString          | UTF-8 字符串处理  |
| sese-event    | github.com/libsese/sese-event       | 网络事件库        |
| sese-plugin   | github.com/libsese/sese-plugin      | 本机插件接口       |

编辑 vcpkg.json 向项目添加依赖

```json
{
  "name": "sese-event-example",
  "version-string": "0.1.0",
  "dependencies": [
    "sese"
  ]
}
```

向 registries -> package 中添加如下库

- simpleuuid
- sstring
- sese-event
- sese-plugin
- sese

## 可用 Target

- Sese::Core

## 日志模块（record）

### 使用静态方法

_record-example-1.cpp_

```clike
#include "sese/record/LogHelper.h"

const char str[] = "Hello World";

int main() {
    sese::record::LogHelper::d("Hello World");
    sese::record::LogHelper::i("STRING: %s", str);
    sese::record::LogHelper::w("NUMBER: %d", 1024);
    sese::record::LogHelper::e("BOOL: %s", true ? "true" : "false");
    return 0;
}
```

```
2023-06-12T11:30:47.978Z D DEF Main:18140> Hello World
2023-06-12T11:30:47.984Z I DEF Main:18140> STRING: Hello World
2023-06-12T11:30:47.989Z W DEF Main:18140> NUMBER: 1024
2023-06-12T11:30:47.993Z E DEF Main:18140> BOOL: true
```

- d、i、w、e 分别代表 debug、info、warn、error 四个级别
- 使用静态方法只能使用默认的 tag —— “DEF”
- 日志参数支持类似 printf 一样的格式

### 使用动态方法

动态方法需要实例化 LogHelper 使用

_record-example-2.cpp_

```clike
#include "sese/record/LogHelper.h"

const char str[] = "Hello World";

int main() {
    sese::record::LogHelper logHelper("MyTag");
    logHelper.debug("Hello World");
    logHelper.info("STRING: %s", str);
    logHelper.warn("NUMBER: %d", 1024);
    logHelper.error("BOOL: %s", true ? "true" : "false");
    return 0;
}
```

```
2023-06-12T11:38:26.054Z D MyTag Main:21172> Hello World
2023-06-12T11:38:26.063Z I MyTag Main:21172> STRING: Hello World
2023-06-12T11:38:26.068Z W MyTag Main:21172> NUMBER: 1024
2023-06-12T11:38:26.072Z E MyTag Main:21172> BOOL: true
```

在需要频繁使用日志器时，使用动态方法会更加具有效率

### 异步日志器

使用异步日志器只需要在 vcpkg features 中添加 <kbd>async-logger</kbd> 即可。

### 日志输出地

_record-example-3.cpp_

!> 不建议使用 BlockAppender，在 block size 较小时存在丢失日志的 bug，问题存在于 0.6.3 版本。

```clike
#include "sese/record/LogHelper.h"
#include "sese/record/Logger.h"
#include "sese/record/BlockAppender.h"

const char str[] = "Hello World";

int main() {
    auto pLogger = sese::record::getLogger();
    auto appender = std::make_shared<sese::record::BlockAppender>(128);
    pLogger->addAppender(appender);

    sese::record::LogHelper logHelper("MyTag");
    logHelper.debug("Hello World");
    logHelper.info("STRING: %s", str);
    logHelper.warn("NUMBER: %d", 1024);
    logHelper.error("BOOL: %s", true ? "true" : "false");
    return 0;
}
```

```
2023-06-12T11:59:12.208Z D MyTag Main:2940> Hello World
2023-06-12T11:59:12.213Z I MyTag Main:2940> STRING: Hello World
2023-06-12T11:59:12.217Z W MyTag Main:2940> NUMBER: 1024
2023-06-12T11:59:12.222Z E MyTag Main:2940> BOOL: true
```

此时产生两个以时间戳命名的文件

_20230612 115912207.log_

```
2023-06-12T11:59:12.208Z D MyTag Main:2940> Hello World
2023-06-12T11:59:12.213Z I MyTag Main:2940> STRING: Hello World
```

_20230612 115912222.log_

```
2023-06-12T11:59:12.217Z W MyTag Main:2940> NUMBER: 1024
2023-06-12T11:59:12.222Z E MyTag Main:2940> BOOL: true
```

## 线程模块（thread）

### 启动一个线程

日志器能够获取到线程模块所属线程的名称，主线程默认为 Main。
对于直接使用 std::thread 未进行处理的情况下会导致获取名称依然为 Main。

_thread-example-1.cpp_

```clike
#include "sese/thread/Thread.h"
#include "sese/record/LogHelper.h"

void ThreadProc() {
    sese::record::LogHelper::d("Hello World");
}

int main() {
    auto th1 = sese::Thread(ThreadProc, "MyThread");
    th1.start();
    th1.join();

    auto th2 = std::thread(ThreadProc);
    th2.join();

    ThreadProc();

    return 0;
}
```

```
2023-06-12T13:35:37.650Z D DEF MyThread:14096> Hello World
2023-06-12T13:35:37.685Z D DEF Main:15840> Hello World
2023-06-12T13:35:37.689Z D DEF Main:16768> Hello World
```

### 启动一个带有参数的线程

_thread-example-2.cpp_

```clike
#include "sese/thread/Thread.h"
#include "sese/record/LogHelper.h"

#include <functional>

void ThreadProc(const char *str) {
    sese::record::LogHelper::d("STRING: %s", str);
}

int main() {
    auto th1 = sese::Thread(std::bind(&ThreadProc, "Hello World"), "MyThread1");
    th1.start();
    th1.join();

    auto th2 = sese::Thread([]() -> void { ThreadProc("Hello World"); }, "MyThread2");
    th2.start();
    th2.join();

    return 0;
}
```

如上所示，可以使用 std::bind 或 lambda 表达式传递参数。（方式不唯一，仅供演示）

```
2023-06-12T13:42:43.916Z D DEF MyThread1:15656> STRING: Hello World
2023-06-12T13:42:43.947Z D DEF MyThread2:6728> STRING: Hello World
```

### 使用线程池

_thread-example-3.cpp_

```clike
#include "sese/thread/ThreadPool.h"
#include "sese/record/LogHelper.h"

#include <functional>
#include <chrono>

using namespace std::chrono_literals;

void ThreadProc(const char *str) {
    sese::record::LogHelper::d("STRING: %s", str);
}

int main() {
    auto pool = sese::ThreadPool("MyThreadPool", 4);
    pool.postTask(std::bind(&ThreadProc, "Hello"));

    std::this_thread::sleep_for(100ms);
    pool.shutdown();
    return 0;
}
```

```
2023-06-12T14:00:46.557Z D DEF MyThreadPool0:21436> STRING: Hello
```

线程池中线程的名称取决于线程池的名称，同时会添加数字后缀

## 开发和调试

开发和调试该项目需要启用额外选项

> -DSESE_BUILD_TEST:BOOL=TRUE
>
> -DVCPKG_MANIFEST_FEATURES:STRING=tests