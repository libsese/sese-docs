# sese-plugin

> sese.plugin 提供可供本机使用的插件接口

## 开始

编辑 vcpkg-configuration.json，向 registries > package 中添加 sese-plugin

编辑 vcpkg.json 向项目依赖中添加依赖

## 查找包并链接到项目中

```cmake
find_package(sese-plugin CONFIG REQUIRED)
target_link_libraries(${TARGET} PRIVATE Sese::Plugin)
```

## 可用 Target

- Sese::Plugin

## 示例 - 自定义插件

### 定制接口

```cpp
#include "sese/plugin/Marco.h"

class Printable : public sese::plugin::BaseClass {
public:
    virtual void run() = 0;
};

class Bye : public Printable {
public:
    void run() override {
        puts("Bye");
    }
};

class Hello : public Printable {
public:
    void run() override {
        puts("Hello");
    }
};
```

### 添加插件信息

```cpp
DEFINE_MODULE_INFO(
        .moduleName = "MyModule",
        .versionString = "0.1.0",
        .description = "The module for test."
)
```

### 注册类

```cpp
DEFINE_CLASS_FACTORY(
        REGISTER_CLASS("com.kaoru.plugin.test.Bye", Bye),
        REGISTER_CLASS("com.kaoru.plugin.test.Hello", Hello)
)
```

## 示例 - 加载插件

> 加载模块相关实例请参考 sese.core 文档