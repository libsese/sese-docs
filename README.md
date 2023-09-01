# sese-docs

> 这里是 libsese 相关基础部件文档集合

## 前置工作

在进一步了解其它仓库前，推荐先了解如何使用我们的 vcpkg-registry，这样你可以比较顺利的导入你需要使用的库

[**点击了解**](./project/vcpkg-registry/README.md)

## 浏览库

### [sese-core](./project/sese-core/README.md)

便捷轻量的跨平台C++开发框架核心

提供 Windows\Linux\macOS 的系统 API 封装，包括但不限于：

- 顺序初始化器
- 常用的配置文件解析
- 网络
- TLS
- 文件
- 日志
- 等

---

### [sese-db](./project/sese-db/README.md)

跨平台的数据库统一接口

数据库支持:

- SQLite
- MariaDB
- MySQL
- PostgreSQL
- 等

---

### [sese-event](./project/sese-event/README.md)

跨平台的网络事件库

支持 WSAEventSelect、Epoll 和 Kqueue 等后端实现。

---

### [sese-plugin](./project/sese-plugin/README.md)

本机二进制插件库，需要配合 sese-core 使用。

---

### [SString](./project/SString/README.md)

UTF-8 字符串处理库。

---

### [SimpleUuid](./project/SimpleUuid/README.md)

分布式唯一 ID 生成器。

---

!> 使用 vcpkg 的目的之一就是希望开发者始终使用最新版本的库，下面是各个库所处于的最新版本

前往 >> [**vcpkg-registry**](https://github.com/libsese/vcpkg-registry/#Packages) <<

## 关于

© 2021-2023 libsese Open Source, All rights reserved.