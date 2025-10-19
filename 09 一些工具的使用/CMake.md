什么是 $CMake$ ？

$CMake$ 是个一个开源的跨平台自动化建构系统。

通过使用简单的配置文件 $CMakeLists.txt$，自动生成不同平台的构建文件，如 $Makefile$、$Visual Studio$ 工程文件等。



# $CMake$ 基础

以 $Tinywebserver$ 为例

```cmake
cmake_minimum_required(VERSION 3.10) # 指定CMake的最低版本要求
project(TinyWebServer LANGUAGES CXX C) # 定义项目名称时指定语言，不指定时默认支持C/C++
# 如需要新增支持语言，可以enable_language(Python)

# 添加头文件搜索路径
include_directories(
	base
	tcp
)

# 设置变量的值 例如 set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "-g -Wall -Werror -std==c++14") # -g 生成调试信息 -Wall 开启编译器警告 -Werror 将所有警告视为错误 
set(CMAKE_CXX_COMPILER "g++")
set(CMAKE_CXX_FLAGS_DEBUG "-O0") # -O0 关闭编译器优化

# 查找库和包
find_package(Threads REQUIRED)

# 将指定目录中的所有源文件（如 .c、.cpp 等）的路径自动添加到变量中
# 方便后续在 add_executable 或 add_library 中引用
aux_source_directory(base SRC_LIST1)
aux_source_directory(tcp SRC_LIST2)

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/build/test/) # 两个都是内置变量 其中PROJECT_SOURCE_DIR是项目的根目录

# 指定要生成的可执行文件和源文件
add_executable(echo_server test/echo_server.cpp ${SRC_LIST1} ${SRC_LIST2})
# 链接目标文件与其他库
target_link_libraries(echo_server ${CMAKE_THREAD_LIBS_INIT})
# 当你使用 find_package(Threads REQUIRED) 查找线程库时，CMake 会根据系统类型（如 Linux、Windows 等）自动检测并确定需要链接的线程库，并将对应的链接信息（例如 Linux 下的 -lpthread，Windows 下的系统线程库标识）存入 CMAKE_THREAD_LIBS_INIT 变量中
```

