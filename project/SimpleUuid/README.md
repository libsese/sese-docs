# SimpleUuid

> SimpleUuid 是一个用于生成分布式 ID 的雪花算法库

## 开始

编辑 vcpkg-configuration.json，向 registries > package 中添加 simpleuuid

编辑 vcpkg.json 向项目中添加依赖

```json
{
  "name": "${PROJECT_NAME}",
  "dependencies": [
    "simpleuuid"
  ]
}
```

## 查找包并链接到项目中

```cmake
find_package(SimpleUuid CONFIG REQUIRED)
target_link_libraries(${TARGET} PRIVATE SimpleUuid-uuid)
```

## 可用 Target

- SimpleUuid::uuid
- SimpleUuid::uuid-static

## 时间回拨处理

uuid::TimestampHandler::tryGetCurrentTimestamp 可以根据不同的情况返回不同的值

- <kbd>0</kbd> 发生时间回拨且回拨时间不超过5秒
- <kbd>UINT64_MAX</kbd> 发生时间回拨且回拨时间超过5秒
- <kbd>other</kbd> 正常时间戳

## 示例 - 生成 UUID

```cpp
#include <chrono>
#include <thread>
#include <iostream>
#include <SimpleUuid/Uuid.h>
#include <SimpleUuid/TimestampHandler.h>

using namespace uuid;

int main() {
    uint8_t selfId = 0x23;
    TimestampHandler handler(std::chrono::system_clock::now());

    for (int i = 0; i < 10; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(2));
        auto uuid = Uuid(selfId, handler.getCurrentTimestamp(), 0x45);
        std::cout << uuid.toNumber() << " "
                  << uuid.getSelfId() << ":"
                  << uuid.getR() << ":"
                  << uuid.getTimestamp()
                  << std::endl;
    }

    return 0;
}
```

输出结果

```
2541439248263215522 #:E:1683542702498
2541439248263215524 #:E:1683542702500
2541439248263215526 #:E:1683542702502
2541439248263215528 #:E:1683542702504
2541439248263215531 #:E:1683542702507
2541439248263215533 #:E:1683542702509
2541439248263215535 #:E:1683542702511
2541439248263215537 #:E:1683542702513
2541439248263215540 #:E:1683542702516
2541439248263215542 #:E:1683542702518
```

## 开发和调试

开发和调试该项目需要启用额外选项

> -DUUID_BUILD_TEST:BOOL=TRUE
>
> -DVCPKG_MANIFEST_FEATURES:STRING=tests