# vcpkg-registry

> 此仓库是 libsese 所属 registry

## 安装 vcpkg

```bash
# 克隆仓库
# ${VCPKG_DEST} 一般取值为
# Unix: /usr/local/vcpkg
# Windows: C:/vcpkg
git clone https://github.com/microsoft/vcpkg.git ${VCPKG_DEST}

# 安装 vcpkg
# Unix
sh ${VCPKG_DEST}/bootstrap-vcpkg.sh 
# Windows
start ${VCPKG_DEST}/bootstrap-vcpkg.bat
```

## 将 vcpkg 应用于项目中

在 CMake 配置命令选项中添加如下语句

```
-DCMAKE_TOOLCHAIN_FILE=${VCPKG_DEST}/scripts/buildsystems/vcpkg.cmake
```

最终的 CMake 命令大致如下

```bash
cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=${VCPKG_DEST}/scripts/buildsystems/vcpkg.cmake
```

!> 具体项目各个参数不一致，请根据你项目的环境和使用的 IDE 进行配置

为你的项目编写 vcpkg.json

```json
{
  "name": "${PROJECT_NAME}",
  "dependencies": []
}
```

## 添加第三方 registry

编辑 vcpkg-configuration.json 文件

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
      "packages": []
    }
  ]
}
```

${BASELINE} 指的是对应 registry 的仓库 commits 的 hash 值

下面的脚本可用于获取 vcpkg 仓库的 BASELINE 值

```bash
# ON ${VCPKG_DEST}
BASELINE=$(git rev-parse HEAD)
echo $BASELINE
```

### 为当前项目添加（推荐）

将该文件存放至当前项目根目录下，即与 vcpkg.json 同级目录

### 全局添加

直接将文件存放至 ${VCPKG_DEST} 下