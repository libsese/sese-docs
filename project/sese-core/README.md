# sese-core

> 跨平台类库

## 开始

本库依赖依赖可移步次[页面](https://github.com/libsese/sese-core?tab=Apache-2.0-1-ov-file)查询

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

## 可用 Target

- Sese::Core
- Sese::Plugin

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
2023-08-03T15:09:00.715Z D LogHelper.cpp:12 Main:13920> Hello World
2023-08-03T15:09:00.715Z I LogHelper.cpp:19 Main:13920> STRING: Hello World
2023-08-03T15:09:00.715Z W LogHelper.cpp:26 Main:13920> NUMBER: 1024
2023-08-03T15:09:00.715Z E LogHelper.cpp:33 Main:13920> BOOL: true
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
    sese::record::LogHelper logHelper;
    logHelper.debug("Hello World");
    logHelper.info("STRING: %s", str);
    logHelper.warn("NUMBER: %d", 1024);
    logHelper.error("BOOL: %s", true ? "true" : "false");
    return 0;
}
```

```
2023-08-03T15:07:57.235Z D LogHelper.cpp:58 Main:6816> Hello World
2023-08-03T15:07:57.235Z I LogHelper.cpp:65 Main:6816> STRING: Hello World
2023-08-03T15:07:57.235Z W LogHelper.cpp:72 Main:6816> NUMBER: 1024
2023-08-03T15:07:57.235Z E LogHelper.cpp:79 Main:6816> BOOL: true
```

在需要频繁使用日志器时，使用动态方法会更加具有效率

### 使用宏

（推荐）原本的宏在不那么久的版本由于设计不合理本身是被移除的，但在<kbd>0.7.6</kbd>版本开始重新可用，这也是我们推荐你使用的方式。

_record-example-4.cpp_

```clike
#include "sese/record/Marco.h"

int main () {
    SESE_DEBUG("Hello World");
    SESE_INFO("STRING: %s", "Hello");
    SESE_WARN("NUMBER: %d", 1024);
    SESE_ERROR("BOOL: %s", true ? "true" : "false");
}
```

根据日志输出，我们可以快速的定位到日志调用的位置。

```
2023-08-03T15:06:54.545Z D record-example-4.cpp:4 Main:14108> Hello World
2023-08-03T15:06:54.545Z I record-example-4.cpp:5 Main:14108> STRING: Hello
2023-08-03T15:06:54.545Z W record-example-4.cpp:6 Main:14108> NUMBER: 1024
2023-08-03T15:06:54.546Z E record-example-4.cpp:7 Main:14108> BOOL: true
```

### 异步日志器

使用异步日志器只需要在 vcpkg features 中添加 <kbd>async-logger</kbd> 即可。

### 日志输出地

_record-example-3.cpp_

!> ~~BlockAppender 在 block size 较小时存在丢失日志的 bug，问题存在于 0.6.3 版本。~~（已修复，block size 最小值限制为 1024
bytes。）

**此处仅作演示，已经不再推荐使用 BlockAppender。**

```clike
#include "sese/record/LogHelper.h"
#include "sese/record/Logger.h"
#include "sese/record/BlockAppender.h"

const char str[] = "Hello World";

int main() {
    auto pLogger = sese::record::getLogger();
    auto appender = std::make_shared<sese::record::BlockAppender>(128);
    pLogger->addAppender(appender);

    sese::record::LogHelper logHelper;
    logHelper.debug("Hello World");
    logHelper.info("STRING: %s", str);
    logHelper.warn("NUMBER: %d", 1024);
    logHelper.error("BOOL: %s", true ? "true" : "false");
    return 0;
}
```

```
2023-08-03T15:09:30.723Z D LogHelper.cpp:58 Main:7336> Hello World
2023-08-03T15:09:30.723Z I LogHelper.cpp:65 Main:7336> STRING: Hello World
2023-08-03T15:09:30.724Z W LogHelper.cpp:72 Main:7336> NUMBER: 1024
2023-08-03T15:09:30.724Z E LogHelper.cpp:79 Main:7336> BOOL: true
```

此时产生两个以时间戳命名的文件

_20230612 115912207.log_

```
2023-08-03T15:09:30.723Z D LogHelper.cpp:58 Main:7336> Hello World
2023-08-03T15:09:30.723Z I LogHelper.cpp:65 Main:7336> STRING: Hello World
```

_20230612 115912222.log_

```
2023-08-03T15:09:30.724Z W LogHelper.cpp:72 Main:7336> NUMBER: 1024
2023-08-03T15:09:30.724Z E LogHelper.cpp:79 Main:7336> BOOL: true
```

## 线程模块（thread）

### 启动一个线程

日志器能够获取到线程模块所属线程的名称，主线程默认为 Main。
~~对于直接使用 std::thread 未进行处理的情况下会导致获取名称依然为 Main。~~
在 0.6.3 之后的版本已修复，未命名线程和非 sese::Thread 创建的线程均定义为 UNKNOWN_THREAD。

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
2023-06-12T13:35:37.685Z D DEF UNKNOWN_THREAD:15840> Hello World
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

### 使用异步 API

_thread-example-4.cpp_

示例演示了使用独立新建线程的异步方法和使用全局线程池的方法。

```clike
#include "sese/util/Initializer.h"
#include "sese/util/Util.h"
#include "sese/thread/Async.h"
#include "sese/record/Marco.h"

int main(int argc, char **argv) {
    sese::initCore(argc, argv);

    SESE_INFO("begin");
    {
        auto future = sese::async<std::string>([]() -> std::string {
            sese::sleep(1s);
            return "Hello, World";
        });
        SESE_INFO("result: %s", future.get().c_str());
    }
    {
        auto future = sese::asyncWithGlobalPool<std::string>([]() -> std::string{
            sese::sleep(1s);
            return "Bye";
        });
        SESE_INFO("result: %s", future.get().c_str());
    }

    return 0;
}
```

输出示例：

```
2024-03-31T10:48:01.102Z I thread-example-4.cpp:9 Main:15992> begin
2024-03-31T10:48:02.108Z I thread-example-4.cpp:15 Main:15992> result: Hello, World
2024-03-31T10:48:03.121Z I thread-example-4.cpp:22 Main:15992> result: Bye
```

## 并发数据结构（concurrent）

此模块已经基本弃用，代码均已标记弃用，请不要继续使用此模块。
模块中数据结构存在一些难以复现的 bug，行为不可预测，如果执意要使用，希望你明白你在做什么。
实现真正的无锁并发的核心思想是尽量减少资源的竞争，而不是不经思考直接使用无锁数据结构。

## 配置模块（config）

> 此模块包含 json、xml 等文件格式的序列化和反序列化，并不是真正意义上的用于“配置”的模块。

### 从流中读取 json 流

_json-example-1.cpp_

```clike
#include "sese/config/json/JsonUtil.h"
#include "sese/util/InputBufferWrapper.h"
#include "sese/record/LogHelper.h"

using namespace sese::json;

int main() {
    const char content[]{
            "{"
            "   \"name\": \"example\","
            "   \"id\": 114514"
            "}"
    };
    auto input = std::make_shared<sese::InputBufferWrapper>(content, sizeof(content));

    auto object = JsonUtil::deserialize(input, 3);
    auto nameObject = object->getDataAs<BasicData>("name");
    auto idObject = object->getDataAs<BasicData>("id");

    sese::record::LogHelper::i(
            "name: %s, id %lld",
            nameObject->getDataAs<std::string>("undef").c_str(),
            idObject->getDataAs<int64_t>(0)
    );

    return 0;
}
```

直接和 JsonObject 打交道或许有些麻烦，可以尝试用位于 Marco.h 下的宏快速操作。

```clike
#include "sese/config/json/JsonUtil.h"
#include "sese/config/json/Marco.h"
#include "sese/util/InputBufferWrapper.h"
#include "sese/record/LogHelper.h"

using namespace sese::json;

int main() {
    const char content[]{
            "{"
            "   \"name\": \"example\","
            "   \"id\": 114514"
            "}"
    };
    auto input = std::make_shared<sese::InputBufferWrapper>(content, sizeof(content));
    auto object = JsonUtil::deserialize(input, 3);

    SESE_JSON_GET_STRING(name, object, "name", "undef");
    SESE_JSON_GET_INTEGER(id, object, "id", 0);

    sese::record::LogHelper::i(
            "name: %s, id %lld",
            name.c_str(),
            id
    );

    return 0;
}
```

结果是一样的：

```
2023-06-16T00:02:21.038Z I DEF Main:17064> name: example, id 114514
```

### 将 json 数据写入流中

```clike
#include "sese/config/json/JsonUtil.h"
#include "sese/util/ConsoleOutputStream.h"

using namespace sese::json;

int main() {
    auto object = std::make_shared<sese::json::ObjectData>();

    auto nameObject = std::make_shared<BasicData>();
    nameObject->setDataAs<std::string>("example");

    auto idObject = std::make_shared<BasicData>();
    idObject->setDataAs<int64_t>(1919810);

    object->set("name", nameObject);
    object->set("id", idObject);

    auto output = std::make_shared<sese::ConsoleOutputStream>();
    JsonUtil::serialize(object, output);

    return 0;
}
```

此实例同样可以使用宏快速完成。

```clike
#include "sese/config/json/JsonUtil.h"
#include "sese/config/json/Marco.h"
#include "sese/util/ConsoleOutputStream.h"

using namespace sese::json;

int main() {
    auto object = std::make_shared<sese::json::ObjectData>();

    SESE_JSON_SET_STRING(object, "name", "example");
    SESE_JSON_SET_INTEGER(object, "id", 1919810);

    auto output = std::make_shared<sese::ConsoleOutputStream>();
    JsonUtil::serialize(object, output);

    return 0;
}
```

结果也是一样的：

```json
{
    "id": 1919810,
    "name": "example"
}
```

### 从流中读取 Yaml

_yaml-example-1.cpp_

```clike
#include <sese/config/yaml/YamlUtil.h>
#include <sese/config/yaml/Marco.h>
#include <sese/util/InputBufferWrapper.h>
#include <sese/record/Marco.h>

int main () {
    const char *str = "root:\n"
                      "  string: \"Hello\"\n"
                      "  int: 90\n"
                      "  bool: yes";
    auto input = sese::InputBufferWrapper(str, strlen(str));
    auto object = std::dynamic_pointer_cast<sese::yaml::ObjectData>(sese::yaml::YamlUtil::deserialize(&input, 3));
    auto root = std::dynamic_pointer_cast<sese::yaml::ObjectData>(object->get("root"));

    SESE_YAML_GET_STRING(valueStr, root, "string", "undef");
    SESE_YAML_GET_INTEGER(valueInt, root, "int", 0);
    SESE_YAML_GET_BOOLEAN(valueBool, root, "bool", false);

    SESE_INFO("str  - %s", valueStr.c_str());
    SESE_INFO("int  - %d", (int) valueInt);
    SESE_INFO("bool - %s", valueBool ? "true" : "false");
}
```

```
2023-8-03T15:35:08.113Z I yaml-example-1.cpp:19 Main:10508> str  - Hello
2023-8-03T15:35:08.114Z I yaml-example-1.cpp:20 Main:10508> int  - 90
2023-8-03T15:35:08.114Z I yaml-example-1.cpp:21 Main:10508> bool - true
```

### 将 Yaml 数据写入流中

_yaml-example-2.cpp_

```clike
#include <sese/config/yaml/YamlUtil.h>
#include <sese/config/yaml/Marco.h>
#include <sese/util/ConsoleOutputStream.h>

int main() {
    auto object = std::make_shared<sese::yaml::ObjectData>();

    SESE_YAML_SET_STRING(object, "STR", "Hello");
    SESE_YAML_SET_BOOLEAN(object, "BOOL", true);
    SESE_YAML_SET_DOUBLE(object, "PI", 3.14);

    sese::ConsoleOutputStream output;
    sese::yaml::YamlUtil::serialize(object, &output);
}
```

```
BOOL: true
PI: 3.140000
STR: Hello
```

### 从流中读取 XML

_xml-example-1.cpp_

```clike
#include "sese/config/xml/XmlUtil.h"
#include "sese/record/LogHelper.h"
#include "sese/util/InputBufferWrapper.h"

int main() {
    const char xml[]{
            "<root>\n"
            "    <person>\n"
            "        <name type=\"string\">foo</name>\n"
            "        <id type=\"integer\">10001</id>\n"
            "    </person>\n"
            "</root>"
    };
    auto input = std::make_shared<sese::InputBufferWrapper>(xml, sizeof(xml));

    auto object = sese::xml::XmlUtil::deserialize(input, 3);
    auto person = object->getElements()[0];
    auto name = person->getElements()[0]->getValue();
    auto nameType = person->getElements()[0]->getAttribute("type", "undef");
    auto id = person->getElements()[1]->getValue();
    auto idType = person->getElements()[1]->getAttribute("type", "undef");

    sese::record::LogHelper::i("value: %s, type: %s", name.c_str(), nameType.c_str());
    sese::record::LogHelper::i("value: %s, type: %s", id.c_str(), idType.c_str());

    return 0;
}
```

```
2023-06-19T16:23:55.725Z I DEF Main:11572> value: foo, type: string
2023-06-19T16:23:55.741Z I DEF Main:11572> value: 10001, type: integer
```

### 将 XML 数据写入流中

_xml-example-2.cpp_

```clike
#include "sese/config/xml/XmlUtil.h"
#include "sese/util/ConsoleOutputStream.h"

using namespace sese::xml;

int main() {
    auto object = std::make_shared<Element>("config");

    auto profile = std::make_shared<Element>("profile");
    object->addElement(profile);

    auto address = std::make_shared<Element>("address");
    address->setAttribute("type", "ipv4");
    address->setValue("192.168.3.1");
    profile->addElement(address);

    auto port = std::make_shared<Element>("port");
    port->setValue("443");
    profile->addElement(port);

    auto output = std::make_shared<sese::ConsoleOutputStream>();
    XmlUtil::serialize(object, output);

    return 0;
}
```

```
<config><profile><address type="ipv4">192.168.3.1</address><port>443</port></profile></config>
```

## 转换模块（convert）

> 此模块负责一些简单的字符串或字节串类的转换

### BASE64 转换

> Base64Converter 在进行编码时，可以手动选择所需码表，可选的有 CodePage::BASE64，CodePage::BASE62，默认则为前者

_convert-example-1.cpp_

```clike
#include "sese/convert/Base64Converter.h"
#include "sese/util/ConsoleOutputStream.h"
#include "sese/util/InputBufferWrapper.h"

int main() {
    auto input = sese::InputBufferWrapper("Hello", 5);
    auto console = sese::ConsoleOutputStream();
    sese::Base64Converter::encode(&input, &console);
    return 0;
}
```

编码的结果如下

```
SGVsbG8=
```

> 解码时则无需指定特定的码表

_convert-example-2.cpp_

```clike
#include "sese/convert/Base64Converter.h"
#include "sese/util/ConsoleOutputStream.h"
#include "sese/util/InputBufferWrapper.h"

int main() {
    auto input = sese::InputBufferWrapper("SGVsbG8=", 8);
    auto console = sese::ConsoleOutputStream();
    sese::Base64Converter::decode(&input, &console);
    return 0;
}
```

解码的结果如下

```
Hello
```

### 百分号编码转换

> 使用百分号编码器编码和解码非 ASCII 字符串

_convert-example-3.cpp_

```clike
#include <sese/convert/PercentConverter.h>
#include <sese/util/FixedBuilder.h>
#include <sese/record/Marco.h>

int main () {
    const char *string = "你好，2023";
    auto buffer1 = std::make_shared<sese::FixedBuilder>(128);
    sese::PercentConverter::encode(string, buffer1);
    buffer1->write("\0", 1);
    SESE_INFO("encode %s", buffer1->data());

    auto buffer2 = std::make_shared<sese::FixedBuilder>(128);
    sese::PercentConverter::decode(buffer1->data(), buffer2);
    buffer2->write("\0", 1);
    SESE_INFO("decode %s", buffer2->data());
    return 0;
}
```

运行结果如下

```
2023-9-01T20:46:18.530Z I convert-example-3.cpp:10 Main:21523> encode %E4%BD%A0%E5%A5%BD%EF%BC%8C%32%30%32%33
2023-9-01T20:46:18.531Z I convert-example-3.cpp:15 Main:21523> decode 你好，2023
```

## 开发和调试

开发和调试该项目需要启用额外选项

> -DSESE_BUILD_TEST:BOOL=TRUE
>
> -DVCPKG_MANIFEST_FEATURES:STRING=tests