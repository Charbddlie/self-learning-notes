# cmake

## CMake语法

CMakeLists.txt中的指令格式是：

    command(args1 args2 …)

command代表不同的命令，args是不同的参数，多参数之间用空格隔开。

## CMake 常用变量

CMAKE_BINARY_DIR、PROJECT_BINARY_DIR、\<projectname\>_BINARY_DIR：指的是工程编译发生的目录

CMAKE_SOURCE_DIR、PROJECT_SOURCE_DIR、\<projectname\>_SOURCE_DIR:  指的是工程顶层目录

CMAKE_CURRENT_SOURCE_DIR:  指的是当前处理的CMakeLists.txt所在路径

CMAKE_CURRENT_BINARY_DIR:  指的是工程编译结果存放的目标目录，可以通过set命令或ADD_SUBDIRECTORY(src bin)改变

这个变量的值，但是set(EXECUTABLE_OUTPUT_PATH \<new_paht\>)并不改变这个变量的值，只会影响最终的保存路径

CMAKE_CURRENT_LSIT_FILE:  指的是当前CMakeLists.txt文件所在完整路径

CMAKE_CURRENT_LSIT_LINE:  指的是调用这个变量当前所在行（在MakeLists.txt中的行数）

CMAKE_MODULE_PATH:  指的是自己加入的cmake模块路径，通过set来设置

EXECUTABLE_OUTPUT_PATH和LIBRARY_OUTPUT_PATH：定义最终编译结果的二进制执行文件和库文件的存放目录

PROJECT_NAME:  指的是通过set设置的PROJECT的名称

ENV{NAME}:  指的是环境变量，通过set(ENV{NAME} new_path)设置，通过$ENV{NAME}调用

CMAKE_INCLUDE_PATH和CMAKE_LIBRARY_PATH：这两个是系统变量而不是cmake变量，需要在bash中用export或在csh中用set命令设置

CMAKE_MAJOR_VERSION： CMAKE主版本号,比如 2.4.6 中的2

CMAKE_MINOR_VERSION： CMAKE次版本号,比如 2.4.6 中的4

CMAKE_PATCH_VERSION： CMAKE补丁等级,比如 2.4.6 中的6

CMAKE_SYSTEM：系统名称,比如 Linux-2.6.22

CMAKE_SYSTEM_NAME： 不包含版本的系统名,比如 Linux

CMAKE_SYSTEM_VERSION： 系统版本,比如 2.6.22

CMAKE_SYSTEM_PROCESSOR： 处理器名称,比如 i686.

UNIX： 在所有的类UNIX平台为 TRUE,包括 OS X 和 cygwin

WIN32：在所有的win32平台为 TRUE,包括 cygwin

## CMake常用指令

cmake_minimum_required：用于说明和设置所需要的CMake的最低版本，一般要在CMakeLists.txt文件开头调用。例如：

cmake_minimum_required(VERSION 3.15.0) 

project：为整个工程设置一个工程名，并可以指定工程支持的语言，默认支持所有语言。例如：

project(OCR CXX C Java)
project(OCR)

cmake系统预定义了两个cmake变量**PROJECT_BINARY_DIR**和**PROJECT_SOURCE_DIR**

对于内部构建（in-source build,在源文件所在目录执行cmake，生成的临时文件和源文件处于同一目录），这两个变量都指向工程所在路径；

对于外部构建（out-of-source build,在源文件之外的其他目录执行cmake），PROJECT_BINARY_DIR指向cmake的编译路径，PROJECT_SOURCE_DIR指向原工程路径。

```cmake
#set：将一个CMAKE变量设置为给定值。例如：

set(VOC_DIR D:\\xxx\\xxx)
```

也可以把一个变量设置成“一系列”值：

```cmake
set(SRC_LIST main.cc util.cc train.cc)

#$: 使用${}的方式对cmake变量取值或引用(if语句中直接使用变量名而不需要通过${}取值)。例如：

set(VOC_DIR D:\\xxx\\xxx)
${VOC_DIR}

#message：在终端向用户输出消息。包含三种类型，SEND_ERROR（产生错误），STATUS（状态信息，带前缀-）和FATAL_ERROR（终止cmake过程），例如：

message(STATUS "This is BINARY dir " ${PROJECT_BINARY_DIR})

#add_executable:  使用指定的源文件（第二个参数，可以一个或多个），生成一个文件名为第一个参数名(这个名称可以任意指定，跟工程名没有关系)的可执行文件。例如：

add_executable(OCR Implement_CTC.cc)
add_executable(OCR Implement_CTC.cc util.cc)
add_executable(OCR ${SRC_LIST})

#CMAKE_INCLUDE_CURRENT_DIR: 添加CMAKE_CURRENT_BINARY_DIR和CMAKE_DURRENT_SOURCE_DIR到当前CMakeLists.txt,相当于INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})的简写
# option: 提供给用户一个可以在命令行进行选择的开关选项，并提供一个默认值(ON 或 OFFF)。例如：

option(BUILD_WITH_CUDA “build project weith CUDA” OFF)
```

冒号里的是描述信息，option和if一起配合使用，可以针对编译环境做不同的定制化处理，包括项目中宏的替换和使用等

```cmake
#configure_file： 将文件复制到指定目录并同时可以选择修改文件属性或特定变量的值。例如：

configure_file("${PROJECT_SOURCE_DIR}/download.sh" "${CMAKE_BINARY_DIR}/download.sh" @ONLY)
configure_file("${PROJECT_SOURCE_DIR}/download.sh" "${CMAKE_BINARY_DIR}/download.sh" COPYONLY)

#add_subdirectory：向当前工程添加存放源文件的子目录（第一个参数）、指定中间二进制和目标二进制文件的存放目录（第二个参数）、指定构建时需要排除的目录（第三个参数），比如example目录是平台相关的，就需要在指定平台上构建时排除掉。例如：

add_subdirectory(src, bin)
```

将src子目录加入工程，并指定编译输出的结果存放路径为bin

```cmake
#与编译结果相关的两个内部变量是EXECUTABLE_OUTPUT_PATH和LIBRARY_OUTPUT_PATH，分别指代可执行文件的输出路径和库的输出路径。可以通过set指令设定这两个路径：

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

#include_directories： 将给定的目录添加到编译器用于搜索包含文件的目录。相对路径则相对于当前源目录。例如：

include_directories(${OPENCV_BUILD_DIR})

#target_include_directories：将给定的目录添加到指定的目标的包含文件的目录。例如：

target_include_directories(OCR PUBLIC ${OpenCV_INCLUDE_DIRS})

#include：包含其他目录的CMakeLists.txt文件。例如：

include(${OPENCV_BUILD_DIR}/CMakeLists.txt)

#find_package：查找并加载外部项目的设置。例如：

find_package(OpenCV REQUIRED)
```

设置的REQUIRED参数表示当找不到 package 时，终止编译过程

```cmake
#add_definitions：为源文件的编译添加由-D引入的宏定义，用于代码中的 “#ifdef xxx #endif”部分。可以同时定义多个，用空格隔开，。例如：

add_definitions(-DWIN32)

#add_dependencies: 定义依赖，需确保在编译前所定义的依赖已经完成构建。
?
#add_library:使用指定的源文件给项目添加一个库。例如：

add_library(OCR ${detecte_src_head})

#target_link_libraries: 将给定的库链接到一个目标上。例如：

target_link_libraries(OCR PUBLIC ${OpenCV_LIBS})

#cmake_policy: 设置策略。 策略是 CMake 中用来改善向后兼容性和追踪兼容性的一种机制，CMake 中的所有策略都被赋予一个 CMPNNNN 格式的识别符，其中 NNNN 是一个整数值。在cmake_policy命令中使用SET可以用来明确地指定一个特定策略是使用新的行为还是旧的行为。例如：

cmake_policy(SET CMP0054 NEW)
cmake_policy(SET CMP0079 OLD)

#aux_source_directory: 查找一个目录中的所有源文件，并将名称列表存储在提供的变量中。例如：

aux_source_directory(. DIR_SRCS)
add_executable(${APP_NAME} ${DIR_SRCS})

#find_file: 查找一个文件的完整路径。例如：

find_file(HELLO_H hello.h)

#set_target_properties： 设置目标的一些属性来改变它们构建的方式。例如：

set_target_properties(ocr PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

#file 查找指定文件，比如*.cpp
#见https://cmake.org/cmake/help/latest/command/file.html#glob-recurse
```