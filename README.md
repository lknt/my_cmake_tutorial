cmake教程

https://www.bilibili.com/video/BV1Z44y1c7UX/?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click&vd_source=5b10a4ae183af6a262a43b53f1974918

仓库https://github.com/franneck94/UdemyCmake







 franneck94 下的拓展，cmake，cmaketool拓展





注意这里要选择visible

![image-20231230153543381](https://github.com/lknt/my_cmake_tutorial/assets/53924313/b5f5b105-5e90-4585-b027-52be06799ea2)

1

写一个cc文件

cmake_minimum_required(VERSION 3.22)

最小的cmake版本号

project(CppProjectTemplate VERSION 1.0.0 LANGUAGE C CXX)

项目名称、版本、语言

```CMAKE
cmake_minimum_required(VERSION 3.22)

project(CppProjectTemplate 
        VERSION 1.0.0 
        LANGUAGES C CXX)

add_executable(Executable main.cc)
```







步骤

0、创建源码和cmake文件

1、mkdir build

2、cd build

3、cmake  -S .. -B 

4、cmake --build .

5、./Executable





```
cmake  -S .. -B 
cmake .. -G "Unix Makefiles"
```

这句才能生成Makefile

如果是Ninja就没有



2_BasicCmakeStructure



构建cmake结构

增加my_lib.cc my_lib.h

在cmake中添加library和targetlink

```
cmake_minimum_required(VERSION 3.22)

project(CppProjectTemplate 
        VERSION 1.0.0 
        LANGUAGES C CXX)

add_library(Library STATIC my_lib.cc)       

add_executable(Executable main.cc)

target_link_libraries(Executable PUBLIC Library)
```





3_IntermidateProject

分离源码和头文件

增加文件夹app和src

我们可以在cmakelist直接加路径

src和app

但是在源文件中不好改



所以选择使用子文件夹



在app和src中都创建cmakelists.txt

然后在主cmakelists.txt中使用add_subdirectory()

在src的cmakelists中增加**target_include_directories(Library "./")**

注意这里的路径是./



把src中的两个my_lib文件放进my_lib文件夹中



记得library的限定词

一个是STATIC，一个是PUBLIC

 ```
add_library(Library STATIC my_lib.cc)

target_include_directories(Library PUBLIC "./")
 ```





4_VariableandOption

主文件中添加

```
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(LIBRARY_NAME Library)
set(EXECUTABLE_NAME Executable)

```

app,src中改成对应的${}



option的使用

可以用if结构控制



可以使用这个命令打开选项

```
cmake .. -DCOMPILE_EXECUTABLE=ON
```



```
cd build
cmake -S .. -B . -G "Unix Makefiles"
cmake .. -G

cmake -S .. -B . -G "Visual Studio 16 2019"


```





5_ConfigureFile

创建configured文件夹

在里面创建cmakelists.txt和config.hpp.in



在config.hpp.in中写

```
#pragma once

#include <cstdint>
#include <string_view>

static constexpr std::string_view project_name = "@PROJECT_NAME@";
static constexpr std::string_view project_version = "@PROJECT_VERSION@";

static constexpr std::int32_t project_version_major{@PROJECT_MAJOR@};
static constexpr std::int32_t project_version_minor{@PROJECT_MINOR@};
static constexpr std::int32_t project_version_patch{@PROJECT_PATCH@};

```

string_view是17的新特性

相当于是只读的字符串



major意味着主要版本，意味着和以前的代码库根本不兼容，有突破性的代码重构，功能增加

minor意味着小版本更新，意味着有功能更新，代码库不变

patch意味着bug修复版本



然后在对应的cmake文件中

```
configure_file(
    "config.hpp.in"
    "${CMAKE_BINARY_DIR}/configured_files/include/config.hpp" ESCAPE_QUOTES
)
```

```
COPYONLY：只拷贝文件，不进行任何的变量替换。这个选项在指定了NEWLINE_STYLE选项时不能使用（无效）。
ESCAPE_QUOTES：躲过任何的反斜杠(C风格)转义。
@ONLY：限制变量替换，让其只替换被@VAR@引用的变量(那么${VAR}格式的变量将不会被替换)。这在配置${VAR}语法的脚本时是非常有用的。
NEWLINE_STYLE style：指定输出文件中的新行格式。UNIX和LF的新行是\n，DOS和WIN32和CRLF的新行格式是\r\n。这个选项在指定了COPYONLY选项时不能使用(无效)。
```



再在主cmake文件中添加subdirectory(configured)

build之后

在build/configure_file里面发现一个hpp文件

就是hpp.in文件对应的一些常量会写入



在lib库的include中添加"${CMAKE_BINARY_DIR}/configured_files/include"

然后就可以在main函数用了



在main函数中使用

#include "config.hpp"

就可以使用config中定义的变量了



6_SourcesAndHeaders

在src和app中设置一些变量存储使用的文件名

```
set(EXE_SOURCES
    "main.cc")
```

```
set(LIBRARY_SOURCES
    "my_lib.cc")
set(LIBRARY_HEADERS
    "my_lib.h")
```



可以使用

cmake --build . --target Executable

make Executable



7_ExternalGit

使用外部库

使用git_submodule



创建external文件夹



git submodule add https://gitee.com/prgrmz00/nlohmann--json.git external/json



发现根目录下有.gitmodules



创建新目录cmake

增加一个.cmake文件

```
function(add_git_submodule dir)
    find_package(Git REQUIRED)

    if (NOT EXISTS ${dir}/CMakeLists.txt)
        execute_process(COMMAND ${GIT_EXECUTABLE}
            submodule update --init --recursive -- ${dir}
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})
    endif()

    add_subdirectory((${dir}))
    
endfunction(add_git_submodule) 
```

如果那个位置不存在cmakelists文件，那么就执行获取子模块的操作

WORKING_DIRECTORY的意思就是在那个文件夹下执行这个命令

否则默认是在binary dir下



设置module的地址（就是有.cmake文件的地址）

并且导入我们定义的cmake的文件

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_PATH}/cmake/")

include(AddGitSubmodule)

并且使用

add_git_submodule(external/json)

这句用以查询这个目录下有无子模块，如果有就只添加子模块进去



在app下的cmake文件添加nlomann的lib

```
target_link_libraries(${EXECUTABLE_NAME} PUBLIC 
    ${LIBRARY_NAME}
    nlohmann_json)
```



我们就可以在main里面使用了



修改cmake库

只有当那个子模块有cmake文件才能加入子目录，否则只能另寻他法

```
if (EXISTS ${dir}/CMakeLists.txt)
        add_subdirectory(${dir})
endif()
```

有时候会有问题

出现error: pathspec '/home/wrf/tests/my_cmake_tutorial/7_ExternalGit/external/json' did not match any file(s) known to git

我也不清楚，只能用这个命令解决

```
git submodule add -f url path/to/submodule
```



当项目没有cmake文件

![image-20231230230338403](https://github.com/lknt/my_cmake_tutorial/assets/53924313/3be10d8b-5635-4027-9752-82f2774b849b)

只能自己去写

![image-20231230230354529](https://github.com/lknt/my_cmake_tutorial/assets/53924313/12791a51-514e-4641-b012-846963479ee9)

dependency graph



在makefile里面

写

dependency:

  cd build && cmake .. --graphviz=graph.dot && dot -Tpng graph.dot -o graphimage.png

这样就能看到一个图片

前提是装了graphviz

![image-20231230231007002](https://github.com/lknt/my_cmake_tutorial/assets/53924313/d9f6e988-b063-4a5f-8245-57861d1f98d8)



8_ExternalLibrariesFetchContent



```
include(FetchContent)

FetchContent_Declare(
        nlohmann_json
        GIT_REPOSITORY https://gitee.com/prgrmz00/nlohmann--json
        GIT_TAG v3.11.3
        GIT SHALLOW True)
Fetch_MakeAvailable(nlohmann_json)
```



GIT SHALLOW True不会递归整个项目

