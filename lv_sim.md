# lvgl_sim

> lvgl pc 仿真

参考: https://docs.lvgl.io/master/details/integration/ide/pc-simulator.html#select-an-ide


## 搭建环境

需要搞定以下环境
- mingw
- cmake
- SDL2

上面两个可以通过`MSYS2`或者手动安装搞定，这里不再细说

安装SDL2
官网：https://www.libsdl.org/
下载 `SDL2-devel-2.30.10-mingw.zip`

解压到软件目录下，记住目录

<!-- 将如下两个目录复制到mingw64的根目录中
SDL2-2.30.1目录中需要复制到mingw64目录中的文件夹：
1）x86_64-w64-mingw32
2）cmake -->



## 搭建工程

LVGL的PC仿真软件：https://github.com/lvgl/lv_port_pc_vscode
下载所有(包括外链的zip)

直接点击根目录下的 `simulator.code-workspace`

`CMakeLists.txt`里添加

```cmake
list(APPEND CMAKE_PREFIX_PATH "D:/SDL2-2.30.10/")
set(SDL2_NO_MWINDOWS 1) # 设置为0的话不显示控制台,也就看不到打印信息
find_package(SDL2 REQUIRED SDL2)
```

然后编译，此时会报错`D:\SDL2-2.30.10\x86_64-w64-mingw32\lib\cmake\SDL2`下的`sdl2-config.cmake`语句出错

修改这三句为自己的路径

```cmake
set(bindir "D:/SDL2-2.30.10/x86_64-w64-mingw32/bin")
set(libdir "D:/SDL2-2.30.10/x86_64-w64-mingw32/lib")
set(includedir "D:/SDL2-2.30.10/x86_64-w64-mingw32/include")
```

CMake可通过，然后编译，编译报错

```
error: call to undeclared library function 'memmove' with type 'void *(void *, const void *, unsigned long long)'; ISO C99 and later do not support implicit function declarations

```

修改`CMakeLists.txt`

```cmake
# Apply additional compile options if the build type is Debug
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    message(STATUS "Debug mode enabled")

    target_compile_options(lvgl PRIVATE
        -pedantic-errors
        -Wall
        -Wclobbered
        -Wdeprecated
        -Wdouble-promotion
        -Wempty-body
        -Wextra
        -Wformat-security
        -Wmaybe-uninitialized
        # -Wmissing-prototypes
        -Wpointer-arith
        -Wmultichar
        -Wno-pedantic # ignored for now, we convert functions to pointers for properties table.
        -Wreturn-type
        -Wshadow
        -Wshift-negative-value
        -Wsizeof-pointer-memaccess
        -Wtype-limits
        -Wundef
        -Wuninitialized
        -Wunreachable-code
        -Wfloat-conversion
        -Wstrict-aliasing
    )


```

为

```cmake
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(lvgl PRIVATE
        -pedantic-errors
        -Wall
        # -Wclobbered 注释掉
        -Wdeprecated
        -Wdouble-promotion
        -Wempty-body
        -Wextra
        -Wformat-security
        # -Wmaybe-uninitialized 注释掉
        # -Wmissing-prototypes
        -Wpointer-arith
        -Wmultichar
        -Wno-pedantic # ignored for now, we convert functions to pointers for propertis table.
        -Wreturn-type
        -Wshadow
        -Wshift-negative-value
        -Wsizeof-pointer-memaccess
        -Wtype-limits
        -Wundef
        -Wuninitialized
        -Wunreachable-code
        -Wfloat-conversion
        -Wstrict-aliasing
        -Wno-implicit-function-declaration #增加该项,不然会报memmove找不到
    )


```


还需要修改`main.c`,不然会报`error: undefined symbol: SDL_main`

```c
#include "lvgl/demos/lv_demos.h"
#include LV_SDL_INCLUDE_PATH //包含SDL的头文件
// #include "glob.h"
```


CMakeLists.txt新增

```cmake
add_custom_command(
    TARGET main  POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
    "${SDL2_DIR}/../x86_64-w64-mingw32/bin/SDL2.dll" ${EXECUTABLE_OUTPUT_PATH}
    )


```

然后编译，链接，搞定

