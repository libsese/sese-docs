# SString

> SString 是用于对 UTF-8 字符串进行简单处理的字符串库

## 开始

编辑 vcpkg-configuration.json，向 registries > package 中添加 sstring

```json
{
  "default-registry": {
    "kind": "git",
    "repository": "https://github.com/microsoft/vcpkg.git",
    "baseline": "${BASELINE}"
  },
  "registries": [
    {
      "kind": "git",
      "repository": "https://github.com/libsese/vcpkg-registry.git",
      "baseline": "${BASELINE}",
      "packages": [
        "sstring"
      ]
    }
  ]
}
```

编辑 vcpkg.json 向项目中添加依赖

```json
{
  "name": "${PROJECT_NAME}",
  "dependencies": [
    "sstring"
  ]
}
```

查找包并链接到项目中

```cmake
find_package(SString CONFIG REQUIRED)
target_link_libraries(${TARGET} PRIVATE SString::SString)
```

## 可用 Target

- SString::String
- SString::SString-static

!> 在 Windows 平台下需要添加 /utf-8 编译选项，否则参与编译的字符串常量会使用非 UTF-8 编码方式

示例

```cmake
target_compile_options(${TARGET} PRIVATE /utf-8)
```

## 示例 - 字符串统计

```cpp
#include <iostream>
#include <SString/SString.h>

using namespace sstr;

int main() {
    auto u8str = SString::fromUTF8("这是一个简单的Demo！");
    auto count = u8str.len();
    auto size = u8str.size();
    auto cap = u8str.cap();
    std::cout << "string len: " << count << std::endl;
    std::cout << "string bytes: " << size << std::endl;
    std::cout << "string cap: " << cap << std::endl;
    return 0;
}
```

输出结果

```
string len: 12
string bytes: 28
string cap: 32
```

## 示例 - 遍历 Unicode

```cpp
#include <iostream>
#include <SString/SString.h>

using namespace sstr;

int main() {
    auto u8str = SString::fromUTF8("这是一个简单的Demo！");
    auto chars = u8str.toChars();
    for (auto ch: chars) {
        printf("\\u%04X", ch.code);
    }
    return 0;
}
```

输出结果

```
\u8FD9\u662F\u4E00\u4E2A\u7B80\u5355\u7684\u0044\u0065\u006D\u006F\uFF01
```

## 示例 - 大小写转换

!> 对于SString(class) 而言，toUpper()、toLower() 等不会改变原本字符串大小的函数将会在原有的基础上修改字符串，而不是产生新的对象

```cpp
#include <iostream>
#include <SString/SString.h>

using namespace sstr;

int main() {
    auto u8str = SString::fromUTF8("这是一个简单的Demo！");
    u8str.toLower();
    std::cout << u8str.data() << std::endl;
    u8str.toUpper();
    std::cout << u8str.data() << std::endl;
    return 0;
}
```

输出结果

```
这是一个简单的demo！
这是一个简单的DEMO！
```

## 示例 - 字符串分割

```cpp
#include <iostream>
#include <SString/SString.h>

using namespace sstr;

int main() {
    auto u8str = SString::fromUTF8("这是一个简单的Demo！");
    auto sub = u8str.substring(2);
    std::cout << sub.data() << std::endl;

    auto list = u8str.split("简单的");
    puts("[");
    bool first = true;
    for (auto item: list) {
        if (first) {
            first = false;
        } else {
            puts(",");
        }
        printf("\t{\"%s\"}", item.data());
    }
    puts("\n]");
    return 0;
}
```

输出结果

```
一个简单的Demo！
[
        {"这是一个"},
        {"Demo！"}
]

```

## 示例 - 组装 UTF-8 字符串

```cpp
#include <iostream>
#include <SString/SStringBuilder.h>

using namespace sstr;

int main() {
    SString str = SString::fromUTF8("你好，");

    SStringBuilder builder(1024);
    builder.append(str);
    builder.append("こんにちは");

    std::cout << builder.toString().data() << std::endl;
    return 0;
}
```

输出结果

```
你好，こんにちは
```

## 开发和调试

开发和调试该项目需要启用额外选项

> -DSSTRING_BUILD_TEST:BOOL=TRUE
> 
> -DVCPKG_MANIFEST_FEATURES:STRING=tests