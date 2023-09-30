# XR806-micro-ROS移植-串口通信

[toc]

## micro-ROS

### 可行性分析

micr-ROS为ROS2专门为微控制器开发的组件，旨在将微控制器变成ROS2的一个节点连接进入ROS系统中。在实际应用中，我们可以实现将微控制器直接接入ROS系统，而不是微控制器首先通过私有协议和树莓派等控制器通信，然后树莓派接入ROS系统的方式。

micr-ROS为其主要针对于中高端的32位单片机，其占用内存根据节点数量和消息类型变化而变化，一般来说，micro-ROS 需要数十 KB 的RAM ，其支持主要硬件如下：

| 板卡                      | MCU                                  | RAM    | flah    |
| ------------------------- | ------------------------------------ | ------ | ------- |
| Espressif ESP32           | ultra-low power dual-core Xtensa LX6 | 520 kB | 4 MB    |
| Raspberry Pi Pico RP2040  | Dual-core Arm Cortex-M0+             | 264 kB | 16 MB   |
| ROBOTIS OpenCR 1.0        | ARM Cortex-M7 STM32F746ZGT6          | 320 kB | 1024 kB |
| Teensy 3.2                | ARM Cortex-M4 MK20DX256VLH7          | 64 kB  | 256 kB  |
| STM32L4 Discovery kit IoT | ARM Cortex-M4 STM32L4                | 128 kB | 1MB     |

详细支持硬件详见[链接1](https://micro.ros.org/docs/overview/hardware/)

支持平台

* FreeRTOS;
* ThreadX;
* Arduino;
*  Zephyr;
* ...

其更详细的支持平台，见[micr-ROS仓库](https://github.com/micro-ROS)[2]

XR806的CPU为Arm-Star ARMv8-M，288KB SRAM，160KB Code ROM. 和16Mbit Flash。从内存上来说，满足要求。另外本次测评的是XR806 FreeRTOS 的SDK，也属于miro-ROS的支持平台之中，加大将miro-ROS移植到XR806中的可能性。

对于外设，一般都使用uart和UDP。XR806也满足要求。

综上，将miro-ROS移植到XR806中，具有可行性；

### micro-ROS框架

![img](https://micro.ros.org/img/micro-ROS_architecture.png)

从框架可以看出，FreeRTOS系统是很容易接入micro-ROS，关于micro-ROS框架的详细描述，请参考官网或Robot Operating System (ROS)这本书籍的

的第七章。

## micro-ROS静态库编译

XR806不在micro-ROS支持的硬件中，因此没有可用静态库使用，但是其官网提供将[micro-ROS编译成静态库](https://micro.ros.org/docs/tutorials/advanced/create_custom_static_library/)[3]的方法。

前提要求：

* 在PC上安装好ROS2，其安装参考[官网](https://docs.ros.org/en/foxy/Installation.html)[4]

也可以使用docker

### 1. 下载micro_ros_setup

micro-ROS相关操作被封装到micro_ros_setup节点中

```shell
# 启动ROS2 环境
source /opt/ros/galactic/setup.sh
# 创建工作空间用于下载micro-ROS tools
mkdir microros_ws
cd microros_ws
git clone -b galactic https://github.com/micro-ROS/micro_ros_setup.git  src/micro_ros_setup
# 更新依赖
sudo apt update && rosdep update
rosdep install --from-paths src --ignore-src -y
# 编译
colcon build
# source 工作空间
source install/local_setup.bash
```

### 2. 下载micro-ROS相关代码

```shell
ros2 run micro_ros_setup create_firmware_ws.sh generate_lib
```

该命令将从github拉取众多程序，保证网络流程，必要是挂梯子才能保证代码拉取成功。若没有梯子，可以在公众号中回复“clash”获取我目前使用的梯子。请耐心等待。

拉取的文件放在`firmware`文件中，具体内容如下

```shell
.
├── COLCON_IGNORE
├── dev_ws
│   ├── ament
│   ├── build
│   ├── install
│   ├── log
│   ├── ros2
│   └── ros2.repos
├── mcu_ws
│   ├── colcon.meta
│   ├── eProsima
│   ├── ros2
│   ├── ros2.repos
│   └── uros
└── PLATFORM

```

### 3. 创建交叉编译链

```shell
touch toolchain.cmake
```

内容如下：

```cmake
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_CROSSCOMPILING 1)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

# SET HERE THE PATH TO YOUR C99 AND C++ COMPILERS
set(CMAKE_C_COMPILER "/home/haijun/xr806/tools/gcc-arm-none-eabi-8-2019-q3-update/bin/arm-none-eabi-gcc")
set(CMAKE_CXX_COMPILER "/home/haijun/xr806/tools/gcc-arm-none-eabi-8-2019-q3-update/bin/arm-none-eabi-g++")


set(CMAKE_C_COMPILER_WORKS 1 CACHE INTERNAL "")
set(CMAKE_CXX_COMPILER_WORKS 1 CACHE INTERNAL "")


# SET HERE YOUR BUILDING FLAGS
set(FLAGS "-O2 -mcpu=cortex-m33  -mtune=cortex-m33 -march=armv8-m.main+dsp -mfpu=fpv5-sp-d16 -mfloat-abi=softfp -mcmse -mthumb --param max-inline-insns-single=500 -DF_CPU=84000000L -D'RCUTILS_LOG_MIN_SEVERITY=RCUTILS_LOG_MIN_SEVERITY_NONE'" CACHE STRING "" FORCE)


set(CMAKE_C_FLAGS_INIT "-std=gnu99 ${FLAGS} -DCLOCK_MONOTONIC=0 -D'__attribute__(x)='" CACHE STRING "" FORCE)
set(CMAKE_CXX_FLAGS_INIT "-std=gnu++98 ${FLAGS} -fno-rtti -DCLOCK_MONOTONIC=0 -D'__attribute__(x)='" CACHE STRING "" FORCE)

set(__BIG_ENDIAN__ 0)

```

* `CMAKE_C_COMPILER`和`CMAKE_CXX_COMPILER`为交叉编译链的路径；

* `FLAGS`设置和芯片相关的宏编译指令，来自于SDK中`gcc.mk`中的设置内容；

### 4. 创建和编译静态库相关的宏定义

```
touch colcon.meta
```

```javascript
{
    "names": {
        "tracetools": {
            "cmake-args": [
                "-DTRACETOOLS_DISABLED=ON",
                "-DTRACETOOLS_STATUS_CHECKING_TOOL=OFF"
            ]
        },
        "rosidl_typesupport": {
            "cmake-args": [
                "-DROSIDL_TYPESUPPORT_SINGLE_TYPESUPPORT=ON"
            ]
        },
        "rcl": {
            "cmake-args": [
                "-DBUILD_TESTING=OFF",
                "-DRCL_COMMAND_LINE_ENABLED=OFF",
                "-DRCL_LOGGING_ENABLED=OFF"
            ]
        }, 
        "rcutils": {
            "cmake-args": [
                "-DENABLE_TESTING=OFF",
                "-DRCUTILS_NO_FILESYSTEM=ON",
                "-DRCUTILS_NO_THREAD_SUPPORT=ON",
                "-DRCUTILS_NO_64_ATOMIC=ON",
                "-DRCUTILS_AVOID_DYNAMIC_ALLOCATION=ON"
            ]
        },
        "microxrcedds_client": {
            "cmake-args": [
                "-DUCLIENT_PIC=OFF",
                "-DUCLIENT_PROFILE_UDP=OFF",
                "-DUCLIENT_PROFILE_TCP=OFF",
                "-DUCLIENT_PROFILE_DISCOVERY=OFF",
                "-DUCLIENT_PROFILE_SERIAL=OFF",
                "-UCLIENT_PROFILE_STREAM_FRAMING=ON",
                "-DUCLIENT_PROFILE_CUSTOM_TRANSPORT=ON"
            ]
        },
        "rmw_microxrcedds": {
            "cmake-args": [
                "-DRMW_UXRCE_MAX_NODES=1",
                "-DRMW_UXRCE_MAX_PUBLISHERS=5",
                "-DRMW_UXRCE_MAX_SUBSCRIPTIONS=5",
                "-DRMW_UXRCE_MAX_SERVICES=1",
                "-DRMW_UXRCE_MAX_CLIENTS=1",
                "-DRMW_UXRCE_MAX_HISTORY=4",
                "-DRMW_UXRCE_TRANSPORT=custom"
            ]
        }
    }
}
```

改文件规定那些文件会被编译，最重要是定义了该静态库许可的最大节点数为1，最大发布量为5，最大订阅量为5，最大服务数为1，。可以根据微控制器的内存调整。

### 5 编译

```shell
ros2 run micro_ros_setup build_firmware.sh $(pwd)/toolchain.cmake $(pwd)/colcon.meta
```

编译成功标志：64个包被编译

```shell
Summary: 64 packages finished [2min 0s]
  40 packages had stderr output: action_msgs actionlib_msgs builtin_interfaces composition_interfaces diagnostic_msgs example_interfaces geometry_msgs libyaml_vendor lifecycle_msgs micro_ros_msgs micro_ros_utilities microxrcedds_client nav_msgs rcl rcl_action rcl_interfaces rcl_lifecycle rcl_logging_interface rcl_logging_noop rclc rclc_lifecycle rclc_parameter rcutils rmw rmw_implementation rmw_microxrcedds rosgraph_msgs rosidl_runtime_c rosidl_typesupport_c rosidl_typesupport_microxrcedds_c sensor_msgs shape_msgs statistics_msgs std_msgs std_srvs stereo_msgs test_msgs trajectory_msgs unique_identifier_msgs visualization_msgs
```

并在`firmware/build`生成静态库`libmicroros.a`和相应的头文件`include`

```shell
haijun@haijun-Lenovo:~/Desktop/microros_ws/firmware/build$ tree -L 2
.
├── include				# 相关头文件
│   ├── actionlib_msgs
│   ├── action_msgs
│   ├── builtin_interfaces
│   ├── composition_interfaces
│   ├── diagnostic_msgs
│   ├── example_interfaces
│   ├── geometry_msgs
│   ├── lifecycle_msgs
│   ├── micro_ros_msgs
│   ├── micro_ros_utilities
│   ├── nav_msgs
│   ├── rcl
│   ├── rcl_action
│   ├── rclc
│   ├── rclc_lifecycle
│   ├── rclc_parameter
│   ├── rcl_interfaces
│   ├── rcl_lifecycle
│   ├── rcl_logging_interface
│   ├── rcutils
│   ├── rmw
│   ├── rmw_microros
│   ├── rmw_microxrcedds_c
│   ├── rosgraph_msgs
│   ├── rosidl_runtime_c
│   ├── rosidl_typesupport_c
│   ├── rosidl_typesupport_interface
│   ├── rosidl_typesupport_introspection_c
│   ├── rosidl_typesupport_microxrcedds_c
│   ├── sensor_msgs
│   ├── shape_msgs
│   ├── statistics_msgs
│   ├── std_msgs
│   ├── std_srvs
│   ├── stereo_msgs
│   ├── test_msgs
│   ├── tracetools
│   ├── trajectory_msgs
│   ├── ucdr
│   ├── unique_identifier_msgs
│   ├── uxr
│   ├── visualization_msgs
│   └── yaml.h
└── libmicroros.a

├── include
│   ├── actionlib_msgs
│   ├── action_msgs
│   ├── builtin_interfaces
│   ├── composition_interfaces
│   ├── diagnostic_msgs
│   ├── example_interfaces
│   ├── geometry_msgs
│   ├── lifecycle_msgs
│   ├── micro_ros_msgs
│   ├── micro_ros_utilities
│   ├── nav_msgs
│   ├── rcl
│   ├── rcl_action
│   ├── rclc
│   ├── rclc_lifecycle
│   ├── rclc_parameter
│   ├── rcl_interfaces
│   ├── rcl_lifecycle
│   ├── rcl_logging_interface
│   ├── rcutils
│   ├── rmw
│   ├── rmw_microros
│   ├── rmw_microxrcedds_c
│   ├── rosgraph_msgs
│   ├── rosidl_runtime_c
│   ├── rosidl_typesupport_c
│   ├── rosidl_typesupport_interface
│   ├── rosidl_typesupport_introspection_c
│   ├── rosidl_typesupport_microxrcedds_c
│   ├── sensor_msgs
│   ├── shape_msgs
│   ├── statistics_msgs
│   ├── std_msgs
│   ├── std_srvs
│   ├── stereo_msgs
│   ├── test_msgs
│   ├── tracetools
│   ├── trajectory_msgs
│   ├── ucdr
│   ├── unique_identifier_msgs
│   ├── uxr
│   ├── visualization_msgs
│   └── yaml.h
└── libmicroros.a				# 静态库

```

到此，静态库的编译工作完成。

## micro-ROS静态库移植

### 1. 复制静态库和头文件

使用工程：`xr806_sdk/project/demo/ros2`

```shell
# 创建microros来存静态文件库和头文件
mkdir -p project/demo/ros2/microros/
# 复制
cp -rf ~/Desktop/microros_ws/firmware/build/ project/demo/ros2/microros/
```

### 2. 将库和头文件包含进工程

修改`project/demo/ros2/gcc/Makefile`

```makefile
...

DIRS_IGNORE += $(ROOT_PATH)/project/common/board/$(shell echo $(CONFIG_BOARD))
# 忽视文件夹中的内容：不参与编译
DIRS_IGNORE +=  $(PRJ_ROOT_PATH)/microros%
...

# extra libraries searching path
PRJ_EXTRA_LIBS_PATH := -L$(PRJ_ROOT_PATH)/microros

# extra libraries
PRJ_EXTRA_LIBS := -lmicroros

# extra header files searching path
PRJ_EXTRA_INC_PATH := -I$(PRJ_ROOT_PATH)/microros/include

...
```

其中`DIRS_IGNORE +=  $(PRJ_ROOT_PATH)/microros%`很重要，不加，按照已有的编译规则，`microros`文件下的内容也会参与编译。

### 3. 添加接口函数

micro-ROS和PC上的ROS服务端通信需要实现`rmw_uros_set_custom_transport`API 中的内容[6]：

```c
rmw_uros_set_custom_transport(
    true, // Framing enabled here. Using Stream-oriented mode.
    (void *) &args,
    my_custom_transport_open,
    my_custom_transport_close,
    my_custom_transport_write,
    my_custom_transport_read
);
```

从定义来说，则是实现uart的初始化，读、写三个功能

创建`set_microros_transports()`函数用于调用`rmw_uros_set_custom_transport()`

```c
static inline void  set_microros_transports()
{
    rmw_uros_set_custom_transport(
                1,
                (void *) NULL,
                serial_transport_open,
                serial_transport_close,
                serial_transport_write,
                serial_transport_read
            );
}
```

实现uart接口函数

```c
/*串口1初始化*/
static int uart_init(void)
{
	UART_InitParam param;

	param.baudRate = 115200;
	param.dataBits = UART_DATA_BITS_8;
	param.stopBits = UART_STOP_BITS_1;
	param.parity = UART_PARITY_NONE;
	param.isAutoHwFlowCtrl = 0;

	if(HAL_UART_Init(UARTID, &param) != HAL_OK)
		return -1;
	/*使能DMA*/
	if (HAL_UART_EnableTxDMA(UARTID) != HAL_OK)
		return -2;
	if (HAL_UART_EnableRxDMA(UARTID) != HAL_OK)
		return -3;
	
	return 0;
}

bool serial_transport_open(uxrCustomTransport* transport){
    HAL_Status status = HAL_ERROR;
    status = uart_init();
    if(status == HAL_OK){
        printf("open uart%d successful!\n", UARTID);
         return 1;
    }
    else{
        printf("open uart%d fail!\n", UARTID);
        return 0;
    }
        
}

bool serial_transport_close(uxrCustomTransport* transport){
    HAL_Status status = HAL_ERROR;
    HAL_UART_DisableTxDMA(UARTID);
    HAL_UART_DisableRxDMA(UARTID);
    status = HAL_UART_DeInit(UARTID);
    if(status == HAL_OK)
        return 1;
    else
        return 0;
}

size_t serial_transport_write(uxrCustomTransport* transport,const uint8_t* buffer,
                            size_t length,uint8_t* errcode){

    return  HAL_UART_Transmit_DMA(UARTID, (uint8_t *)buffer, length);
}

size_t serial_transport_read(uxrCustomTransport* transport,uint8_t* buffer,
                            size_t length,int timeout,uint8_t* errcode){
    
    return HAL_UART_Receive_DMA(UARTID, buffer, length, timeout); 
}

#endif

```

### 4. 添加时钟同步函数

micro-ROS还需要一个时钟函数，以保证微控制器和ROS2同步

```c
#define micro_rollover_useconds 4294967295

int clock_gettime(clockid_t unused, struct timespec *tp)
{
    (void)unused;
    static uint32_t rollover = 0;
    static uint32_t last_measure = 0;

    uint32_t m = OS_GetTicks() * 1000;
    

    rollover += (m < last_measure) ? 1 : 0;

    uint64_t real_us = (uint64_t) (m + rollover * micro_rollover_useconds);

    tp->tv_sec = real_us / 1000000;
    tp->tv_nsec = (real_us % 1000000) * 1000;
    last_measure = m;
    return 0;
}

```

时间计算主要通过滴答函数`OS_GetTicks()`获取。

到此，完成micro-ROS的移植！

## 测试

### publish测试

XR806发布一个int32位的数字，在ROS2上检验是否发布有相应的节点，话题，是否查询到话题的消息。

#### XR806端

XR806端publish代码：

```c
#include <rcl/rcl.h>
#include <rcl/error_handling.h>
#include <rclc/rclc.h>
#include <rclc/executor.h>

static rcl_publisher_t publisher;
static std_msgs__msg__Int32 msg;

static rclc_executor_t executor;
static rclc_support_t support;
static rcl_allocator_t allocator;

static rcl_node_t node;
static rcl_timer_t timer;

#define RCCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){printf("Failed status on line %d: %d. Aborting.\n",__LINE__,(int)temp_rc); return;}}
#define RCSOFTCHECK(fn) { rcl_ret_t temp_rc = fn; if((temp_rc != RCL_RET_OK)){printf("Failed status on line %d: %d. Continuing.\n",__LINE__,(int)temp_rc);}}

static void timer_callback(rcl_timer_t * timer, int64_t last_call_time)
{  
    if (timer != NULL) 
    {
        RCSOFTCHECK(rcl_publish(&publisher, &msg, NULL));
        msg.data++;
         printf("msg.data = %d\n",  msg.data);
    }
    else {
        printf("micrros timer null\n");
    }
}

void microros_pub_int32()
{
    OS_MSleep(100);
    RCCHECK(rclc_executor_spin_some(&executor, RCL_MS_TO_NS(100)));

}

void microros_pub_int32_init()
{
    #if defined MICROROS_SERIAL
    set_microros_transports();
    #endif

    allocator = rcl_get_default_allocator();

    //create init_options
    RCCHECK(rclc_support_init(&support, 0, NULL, &allocator));
    // create node
    RCCHECK(rclc_node_init_default(&node, "xr806_node", "", &support));
    // create publisher
    RCCHECK(rclc_publisher_init_default(
      &publisher,
      &node,
      ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32),
      "xr806_node_publisher")
      );
    // create timer
    const unsigned int timer_timeout = 1000;
    RCCHECK(rclc_timer_init_default(
      &timer,
      &support,
      RCL_MS_TO_NS(timer_timeout),
      timer_callback)
      );
    // create executor
    RCCHECK(rclc_executor_init(&executor, &support.context, 1, &allocator));
    RCCHECK(rclc_executor_add_timer(&executor, &timer));

    msg.data = 0;
    printf("micro_ros init successful.\n");
}
```

设置节点：xr806_node；话题xr806_node_publisher，消息类型：std_msgs.msg.Int32。并定义了一个1000ms的定时器，用于`msg.data++`。

在主函数中初始化`microros_pub_int32_init();`,循环调用`microros_pub_int32();`

#### PC端

创建micro-ROS agent

```shell
source /opt/ros/galactic/setup.sh

ros2 run micro_ros_setup create_agent_ws.sh
ros2 run micro_ros_setup build_agent.sh

source install/local_setup.sh 
```

在PC端运行agent

```shell
ros2 run micro_ros_agent micro_ros_agent serial -D /dev/ttyUSB1 -v6
```

启动XR806，结果如下





## 参考

[1] https://micro.ros.org/docs/overview/hardware/

[2] https://github.com/micro-ROS

[3] https://micro.ros.org/docs/tutorials/advanced/create_custom_static_library/

[4] https://docs.ros.org/en/foxy/Installation.html

[6] https://micro.ros.org/docs/tutorials/advanced/create_custom_transports/