# 基于cmake构建gd32f470工程

[toc]

![大纲](../picture/基于cmake构建gd32f470工程/大纲.png)

对于cortex-M核的MCU，主流还是使用keil或者IAR进行代码编写、编译和调试。美中不足的就是它们在window系统中使用，无法在linux环境中使用。今天就在linux下使用cmake+vsode实现gd32f470的编程。

本次目标：

1. 基于cmake能编译出elf文件；
2. 基于pyocd将elf文件下载到gd32f470中；

## 准备工作

1. MCU：立创的梁山派，mcu为gd32f470zg；
2. 下载器：DAP-link，梁山派自带；
3. PC：ubuntu22.4;
4. 编辑器：vscode;
5. gd32f4的HAL库：GD32F4xx_Firmware_Library_V3.2.0，下载地址：gd32官网；
6. stm32cubemx-lin-v6-11-0：因为gd332提供的HAL只有keil和IAR的编译方式，无gcc的编译。但是呢，gd32基本上那个能无缝使用STM32的程序，刚好stm32cubemx生成的代码有cmake的编译方式，所以得用stm32cubemx生成一个基于cmake构建的工程，拿到cmake的框架；
7. gcc：gcc-arm-none-eabi-10-2020-q4-major：gccd的版本没有什么要求，有一个gcc交叉编译链就行；

以上为大致需要准备的硬件和软件。

要实现基于cmake构建gd32f470工程，的难点在于：

1. 一个比较完备的cmake框架，得站在巨人的肩膀上，幸好stm32cubemx能帮我提供。不过对于嵌入式开发来说，基于makefile来构建是最常见的，但是我觉得复杂，非必须不使用。
2. 对于MCU来说，启动文件是必须的，因为MCU一般都不是从地址0启动，中断向量表还得设置。所一般厂家会提供IAR、ARM、GCC的汇编启动文件。可惜gd32不提供GCC的启动文件；
3. 链接文件：这个由于编译器的不同，有不同的后缀。在IAR中为`.icf`、keil中为`.sct`、在gcc中一般为`.ld`。它们的来源为gd32提供在在线补丁包`GD32F4xx_AddOn_V3.2.0`中。其中`IAR_GD32F4xx_ADDON.3.1.0.exe`为IAR提供，`GigaDevice.GD32F4xx_DFP.3.2.0.pack`为keil提供。可以没有GCC相关的。

## 获取启动文件和链接文件

我们知道rtthread是支持GCC编译的，所以去rtthread的git仓库大概率能找到这两个文件。搜索发现rtthread是支持梁山派的，那就可以直接拿过来用

链接文件：`bsp/gd32/arm/gd32470z-lckfb/board/linker_scripts/link.ld`

启动文件：`bsp/gd32/arm/libraries/GD32F4xx_Firmware_Library/CMSIS/GD/GD32F4xx/Source/GCC/startup_gd32f4xx.s`

## 实现cmake编译

### 获取cmake模板

直接在stm32cubemx新建一个stm32f4的工程，然后构建方式选择cmake。除去源码，可以得到以下模板：

```bash

├── cmake
│   ├── gcc-arm-none-eabi.cmake	# 和交叉编译链相关的配置
│   └── stm32cubemx
│       └── CMakeLists.txt		# 将STM32的相关库编译成静态库，供顶层调用
└── CMakeLists.txt				# 顶层CMakeLists
```

文件不多，就3个

- `gcc-arm-none-eabi.cmake`

实现引入交叉编译链，设置编译参数、设置链接文件等功能。

核心代码（大概率会改动的）如下：

```cmake
set(TOOLCHAIN_PREFIX                arm-none-eabi-) #设置交叉编译链路径
set(TARGET_FLAGS "-mcpu=cortex-m4 -mfpu=fpv4-sp-d16 -mfloat-abi=hard ") #设置MCU内核，硬浮点，本次这个是不用改动的
set(CMAKE_C_LINK_FLAGS "${CMAKE_C_LINK_FLAGS} -T \"${CMAKE_SOURCE_DIR}/STM32F429ZGTx_FLASH.ld\"")	# 设置链接文件

```

- `stm32cubemx --> CMakeLists.txt`

核心代码（大概率会改动的）如下：

```cmake
# 设置宏定义
target_compile_definitions(stm32cubemx INTERFACE 
	USE_HAL_DRIVER 
	STM32F429xx
    $<$<CONFIG:Debug>:DEBUG>
)
# 设置头文件
target_include_directories(stm32cubemx INTERFACE
	...
)
# 设置源文件
target_sources(stm32cubemx INTERFACE
	...
）
```

- `CMakeLists.txt`

```cmake
# 包含gcc-arm-none-eabi.cmake
include("cmake/gcc-arm-none-eabi.cmake")
# 包含子目录下的CMakeLists
add_subdirectory(cmake/stm32cubemx)
# 链接库
target_link_libraries(${CMAKE_PROJECT_NAME}
    stm32cubemx
)
```

有了这个模块，移植到gd32工程中就要简单多了，大致工作为添加交叉编译路径、添加头文件、添加源文件这三步了。

## 构建gd32f470工程

### 整理工程目录

参照keil的工程目录，整理的目录如下：

```shell
├── 3rdPary			# 第三方库，目前未使用
│   └── RTT
├── build			# 存放cmake产生的编译文件
├── CMakeLists.txt	# 顶层CMakeLists
├── Firmware		# 存放gd提供的HAL库
│   ├── CMakeLists.txt	# Firmware子目录下的CMakeLists
│   ├── CMSIS
│   ├── GD32F4xx_standard_peripheral
│   ├── link.ld		# 从rtthread中获取的链接文件
│   └── startup_gd32f4xx.s	# 从rtthread中获取的启动文件
├── gcc-arm-none-eabi.cmake		# 交叉编译链相关配置
├── Hardware
│   ├── CMakeLists.txt		# Hardware子目录下的CMakeLists
│   └── LED					# LED的配置文件，用于指示程序是否下载成功
├── pyocd					# pyocd相关的配置文件
│   ├── GigaDevice.GD32F4xx_DFP.3.2.0.pack
│   └── pyocd.yaml
└── user			
    ├── CMakeLists.txt		# user子目录下的CMakeLists.txt
    ├── gd32f4xx_it.c
    ├── gd32f4xx_it.h
    ├── gd32f4xx_libopt.h
    ├── main.c
    ├── main.h
    ├── systick.c
    └── systick.h

```

顶层CMakeLists.txt来自于模板中的顶层CMakeLists.txt；子目录下的CMakeLists.txt来自于`cmake/CMakeLists.txt`。`gcc-arm-none-eabi.cmake`原封不动拿过来。

### 实现cmake代码

下面就开始补充cmake相关代码（只显示需要修改的代码）

* `gcc-arm-none-eabi.cmake`

```shell
# 交叉编译路径
set(TOOLCHAIN_PREFIX /home/haijun/software/gcc-arm-none-eabi/gcc-arm-none-eabi-10-2020-q4-major/bin/arm-none-eabi-)
# 链接ld文件
set(CMAKE_C_LINK_FLAGS "${CMAKE_C_LINK_FLAGS} -T \"${CMAKE_SOURCE_DIR}/Firmware/link.ld\"")
```

`{CMAKE_SOURCE_DIR}`为cmake预留的变量，为工程路径。

- `Firmware子目录下的CMakeLists.txt`

```shell
# 设置宏定义
target_compile_definitions(gd32f470HAL INTERFACE 
    USE_STDPERIPH_DRIVER
    GD32F470
    $<$<CONFIG:Debug>:DEBUG>
)
# 头文件
target_include_directories(gd32f470HAL INTERFACE
    ${CMAKE_SOURCE_DIR}/Firmware/GD32F4xx_standard_peripheral/Include
    ${CMAKE_SOURCE_DIR}/Firmware/CMSIS
    ${CMAKE_SOURCE_DIR}/Firmware/CMSIS/GD/GD32F4xx/Include
)
# 递归查找Firmware文件夹下所有C文件
file(GLOB_RECURSE SRC_DIR_LIST ${CMAKE_SOURCE_DIR}/Firmware/*.c)
# 源文件
target_sources(gd32f470HAL INTERFACE
    ${SRC_DIR_LIST}
    ${CMAKE_SOURCE_DIR}/Firmware/startup_gd32f4xx.s
)
```

使用`file`递归查找了`Firmware`下所有c文件。

在源文件时，把会变的启动代码也添加进去了

- `Hardware子目录下的CMakeLists.txt`

```shell
# 头文件
target_include_directories(hardware INTERFACE
    ${CMAKE_SOURCE_DIR}/Hardware/LED
)
# 递归查找Hardware文件夹下所有C文件
file(GLOB_RECURSE SRC_DIR_LIST ${CMAKE_SOURCE_DIR}/Hardware/*.c)

# 源文件
target_sources(hardware INTERFACE
    ${SRC_DIR_LIST}
)
```

和上面的类似，指明头文件和源文件;

- `user子目录下的CMakeLists.txt`

```shell
# 头文件
target_include_directories(user INTERFACE
    ${CMAKE_SOURCE_DIR}/user
)
# 递归查找user文件夹下所有C文件
file(GLOB_RECURSE SRC_DIR_LIST ${CMAKE_SOURCE_DIR}/user/*.c)

# 源文件
target_sources(user INTERFACE
    ${SRC_DIR_LIST}
)
```

和上面的类似，指明头文件和源文件;

### 基于cmake编译

```shell
cd build
cmake ..
```

很不幸出现一个错误

```shell
/home/haijun/software/gcc-arm-none-eabi/gcc-arm-none-eabi-10-2020-q4-major/bin/../lib/gcc/arm-none-eabi/10.2.1/../../../../arm-none-eabi/bin/ld: CMakeFiles/gd32f470BaseProject.dir/Firmware/startup_gd32f4xx.s.obj: in function `startup_enter':
/home/haijun/桌面/gd32f470_project_cmake/Firmware/startup_gd32f4xx.s:166: undefined reference to `entry'
```

从报错知道，在`startup_gd32f4xx.s`中没有找到`entry`

通过查看代码：

```
startup_enter:
    bl SystemInit
    bl entry
```

这是汇编转C的代码，`SystemInit`转到时钟配置源码处，通过查rtthread的`entry`函数，知道它是转入rtthread的入口函数。本次为裸机，那就应该转到`main`函数入口

所以修改如下：

```
startup_enter:
    bl SystemInit
    bl main
```

重新编译，得到elf文件

```shell
[  5%] Linking C executable gd32f470BaseProject.elf
Memory region         Used Size  Region Size  %age Used
            CODE:        1944 B         1 MB      0.19%
            DATA:         524 B       448 KB      0.11%
[100%] Built target gd32f470BaseProject
```

## 下载

下载使用pyocd

pyocd的使用前看上一篇推文。

利用cmake的`add_custom_target`命令，将pyocd的下载集中到cmake中

```cmake
# 自定义下载程序命令
add_custom_target( download 
    COMMAND echo "download ${CMAKE_PROJECT_NAME}.elf by pyocd!"
    COMMAND pyocd load ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.elf
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/pyocd
)
```

执行：`make download`

相当于在终端执行`pyocd load ${CMAKE_PROJECT_NAME}.elf --target gd32f470zg --pack ${CMAKE_SOURCE_DIR}/pyocd/GigaDevice.GD32F4xx_DFP.3.2.0.pack`

为了方便清除编译结果，自定义了一段清除代码

```cmake
# 自定义清除build文件命令
add_custom_target( clc
    COMMAND echo " clean ${CMAKE_BINARY_DIR} file!"
    COMMAND rm -rf ${CMAKE_BINARY_DIR}/*
)
```

执行：`make clc`

## 解决vscode找不到函数的报错

vscode提示找不到函数、宏定义的问题，是由于没有告诉vsode头文件路径的原因。

在`.vscode`文件夹中新建`c_cpp_properties.json`，为其添加搜索的头文件

```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "${workspaceFolder}/user/**",
                "${workspaceFolder}//Hardware/LED/**",
                "${workspaceFolder}/Firmware/GD32F4xx_standard_peripheral/Include/**",
                "${workspaceFolder}/Firmware/CMSIS/**",
                "${workspaceFolder}/Firmware/CMSIS/GD/GD32F4xx/Include/**"
                
            ],
            "defines": [
                "GD32F470",
                "USE_STDPERIPH_DRIVER"
            ],
            // "compilerPath": "/usr/bin/gcc",
            "cStandard": "c11",
            "cppStandard": "gnu++11"
            // "intelliSenseMode": "linux-gcc-x64",
            // "compileCommands": "${workspaceFolder}/build/compile_commands.json"
        }
    ],
    "version": 4
}
```

现在就可以基于cmake在vsode中愉快的编程了。

## 总结

由于借助了stm32cubemx和rtthread，所以整体流程比较简单，满足了我在linux上开发的基本需求。





