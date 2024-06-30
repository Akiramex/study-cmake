# 第一课

## CMakeList.txt

~~~cmake
cmake_minimum_required(VERSION 3.12)
project(hellocmake LANGUAGES CXX)

// 生成静态库
add_library(hellolib STATIC hello.cpp)
{
	STATIC->SHARE可生成动态库
}
// 编译main.cpp生成a.out可执行文件
add_executable(a.out main.cpp)

// a.out连接生成的静态库
target_link_libraries(a.out PUBLIC hellolib)
~~~



静态库：相当于把静态库的代码插入到可执行文件中，不需要而外依赖

动态库：在需要用到的时候**重定向**到动态库文件内，Windows下，优先查找可执行文件的同一目录内，其次PATH



~~~cmake
add_subdirectory(hellolib)  // 添加子模块

// 添加搜索目录，这样处理之后甚至可以用<>引用，target_include_directories()指定的路径被视为与系统路径等价
target_include_directories(a.out PUBLIC 目录) 
~~~





~~~cmake
// 在子模块添加搜索路径. 其他引用该模块的Cmake会自动添加这个路径

add_library(hellolib STATIC hello.cpp)
target_include_directories(a.out PUBLIC .) 
~~~





~~~cmake
target_add_definitions(myapp PUBLIC MY_MACRO=1)  # 添加一个宏定义
target_add_definitions(myapp PUBLIC -DMY_MACRO=1)  # 等价 MY_MACRO=1

target_compile_options(myapp PUBLIC -fopenmp)	# 添加编译器命令行选项

target_sources(myapp PUBLIC hello.cpp other.cpp) # 添加要编译的源文件
~~~



~~~cmake
// 下面的指令，把选项添加到接下来的所有目标去

include_directories(/opt/cuda/include)	# 添加头文件搜索目录
link_directories(/opt/cuda)				# 添加库文件的搜索路径
add_definitions(MY_MACRO=1)				# 添加一个宏定义
add_compile_options(-fopenmp)			# 添加编译器命令行选项
~~~



## 三方库

### 纯头文件引入

1. nothings/stb  - 大名鼎鼎的 stb_image系列，涵盖图像，声音，字体等
2. Tencent/rapidjson - 单纯的JSON库
3. gabime/spdlog - 适配控制台，安卓等多后端的日志库

优点：只需要把他们的include目录或头文件下载下来，然后include_directories(spdlog/include)即可。

缺点：函数直接实现在头文件里，没有提前编译，从而需要重复编译同样的内容，编译时间长

### 作为子模块引入

~~~cmake
add_subdirectory(fmt)	
~~~



### 引用系统中预安装的第三方库

~~~cmake
find_package(TBB REQUIRED COMPONENTS tbb tbbmalloc REQUIRED)
target_link_libraries(myexec PUBLIC TBB::tbb TBB::tbbmalloc)
~~~



# 第二课

~~~cmake
#一个标准的CMakeLists.txt模板
cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(projiectName LANGUAGES C CXX)

if (PROJECT_BINARY_DIR STREQUAL PRJECT_SOURCE_DIR)
	message(WARNING "The binary directory of CMake cannot be the same as source directory!")
endif()

if (NOT CMAKE_BUILD_TYEP)
	set(CMAKE_BUILD_TYPE Release)
endif()

if (WIN32)
	add_definitions(-DNOMINMAX -D_USE_MATH_DEFINES)
endif()

if (NOT MSVC)
	find_program(CCACHE_PROGRAM ccache)
	if (CCACHE_PROGRAM)
		message(STATUS "Found CCache: ${CCACHE_PROGRAM}")
		set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ${CCACHE_PROGRAM})
		set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ${CCACHE_PROGRAM})
	endif()
endif()
~~~



# 第三课

## 推荐的目录组织方式

+ 项目名/include/项目名/模块名.h
+ 项目名/src/模块名.cpp



+ CMakeList中写：
+ target_include_directories(项目名 PUBLIC include)



+ 源码文件中写：
+ #include <项目名/模块名.h>
+ 项目名::函数名();



+ 头文件（项目名/include/项目名/模块名.h）中写：

~~~cpp
#pragma one
namespace 项目名 {
    void 函数名();
}
~~~

+ 实现文件（项目名/src/模块名.cpp）中写：

~~~cpp
#include <项目名/模块名.h>
namespace 项目名 {
    void 函数名(){ 函数实现 }
}
~~~



~~~bash
$ tree
.
|-biology
|	|-CMakeList.txt
|	|-include
|	|	|-biology
|	|		|-Animal.h
|	|-src
|		|-Animal.cpp
|-CMakeLists.txt
|-pybmain
	|-CMakeLists.txt
	|-include
	|	|-pybmain
	|		|-myutils.h
	|-src
		|-main.cpp
~~~



# 课件



 [cmake_slides 1.pptx](ppt\cmake_slides 1.pptx)  [cmake_slides 2.pptx](ppt\cmake_slides 2.pptx)  [cmake_slides 3.pptx](ppt\cmake_slides 3.pptx) 



# 来源

https://github.com/parallel101/course
