---
{"aliases":null,"tags":null,"source":null,"created":"2023-03-08 16:15:17","updated":"2023-03-08 16:15:17","uid":null,"title":"vcpkg 安装使用","dg-publish":true,"permalink":"/Pages/vcpkg 安装使用/","dgPassFrontmatter":true,"noteIcon":""}
---


# vcpkg 安装使用

vcpkg 是一款免费的 C/C++ 包管理器，可以用来获取和安装 C/C++依赖包

## 安装 vcpkg

[Get started with vcpkg](https://vcpkg.io/en/getting-started.html)
```
git clone https://github.com/Microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh
```

## 安装项目依赖的库

```
vcpkg install [packages to install]
```

在 Cmake 中使用 vcpkg
```
cmake -B [build directory] -S . -DCMAKE_TOOLCHAIN_FILE=[path to vcpkg]/scripts/buildsystems/vcpkg.cmake
```

然后使用一下命令编译
```
cmake --build [build directory]
```

## 示例

在项目中使用 `sqlite3` 库
```
❯ ./vcpkg install sqlite3
Computing installation plan...
The following packages are already installed:
    sqlite3[core]:x64-linux -> 3.37.2#1
sqlite3:x64-linux is already installed
Restored 0 packages from /home/aiyolo/.cache/vcpkg/archives in 1.303 us. Use --debug to see more details.

Total elapsed time: 1.969 ms

sqlite3 provides CMake targets:
    # this is heuristically generated, and may not be correct
    find_package(unofficial-sqlite3 CONFIG REQUIRED)
    target_link_libraries(main PRIVATE unofficial::sqlite3::sqlite3)
```

最后几行显示了如何在项目中引用 sqlite 库

下面的 main.cpp 会输出 sqlite 的版本号
```
// main.cpp
#include <sqlite3.h>
#include <stdio.h>

int main()
{
    printf("%s\n", sqlite3_libversion());
    return 0;
}
```

在 CMakeLists.txt 写入一下内容
```
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(test)

find_package(unofficial-sqlite3 CONFIG REQUIRED)

add_executable(main main.cpp)

target_link_libraries(main PRIVATE unofficial::sqlite3::sqlite3)
```

使用以下命令进行编译

```
10140  mkdir build
10141  cd build
10142  cmake .. -DCMAKE_TOOLCHAIN_FILE=../vcpkg/scripts/buildsystems/vcpkg.cmake
10143  cmake --build .
10144  ./main
```

## References
- [Getting Started with Classic mode | Microsoft Learn](https://learn.microsoft.com/zh-cn/vcpkg/examples/installing-and-using-packages)