# XR806-micro-ROS移植

[toc]

## udp 接口实现

需要实现的接口

```c
rmw_uros_set_custom_transport(
                false,
                (void *) NULL,
                udp_transport_open,
                udp_transport_close,
                udp_transport_write,
                udp_transport_read
            );
```

### 1. udp:open

安装服务端思路初始化udp

```c
/*UDP初始化*/
int16_t udpclient_init(){
	/*创建一个socket*/
	if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) == -1){
		printf("Socket error\n");
		return -1;
	}
	 /* 初始化预连接的服务端地址 */
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT);
	server_addr.sin_addr.s_addr = inet_addr(REMOTE_IP);
	memset(&(server_addr.sin_zero), 0, sizeof(server_addr.sin_zero));
	/*连接*/
	if(connect(sock, (struct sockaddr *)&server_addr, sizeof(struct sockaddr)) < 0){
		printf("Connect <%d> fail!\n", sock);
        return -2;
	}
    printf("Connect %d sucessful!\n", sock);
  
	return 1;
}

bool udp_transport_open(uxrCustomTransport* transport){

    if(udpclient_init())
    {
        printf("open udp successful!\n");
        return 1;
    }
    else{
        printf("open udp fail!\n");
        printf("remote IP: %s/t port: %d!", REMOTE_IP, PORT);
        return 0;
    }      
}
```

### 2 udp:close

关闭socket

```c
bool udp_transport_close(uxrCustomTransport* transport){

    return closesocket(sock);
}
```

### 3 udp: write

调用socket的`send`发送数据

```c
size_t udp_transport_write(uxrCustomTransport* transport,const uint8_t* buffer,
                            size_t length,uint8_t* errcode){

    size_t rv = 0;
    ssize_t bytes_sent = send(sock, (void*)buffer, length, 0);
    if (-1 != bytes_sent)
    {
        rv = (size_t)bytes_sent;
        *errcode = 0;
    }
    else
    {
        *errcode = 1;
    }
    return rv;
}
```

### 4 UDP : read

在调用`recv`接收数据，由于`recv`常规设置为阻塞函数，不满足`udp_transport_read()`要求读函数为一个超时函数，因此使用`select()`设置超时时间，使`recv`变成非阻塞函数。

```c
size_t udp_transport_read(uxrCustomTransport* transport,uint8_t* buffer,
                            size_t length,int timeout,uint8_t* errcode){
    
    size_t rv = 0;
    
    timeout = (timeout <= 0) ? 1 : timeout;

    struct timeval tv;
    tv.tv_sec = timeout / 1000;
    tv.tv_usec = (timeout % 1000) * 1000;

    fd_set readfds;
    FD_ZERO(&readfds);
    FD_SET(sock, &readfds);
    select(sock+1, &readfds, NULL, NULL, &tv);
    
    if(FD_ISSET(sock, &readfds))
    {
        size_t bytes_received = recv(sock, (void*)buffer, length, 0);
        if (-1 != bytes_received)
        {
            rv = (size_t)bytes_received;
            *errcode = 0;
        }
        else
        {
            *errcode = 1;
        }
    }
    else{
        *errcode = 1;
        return 0;
    }

    return rv;
}
```



## 使用

### 1 code

下载SDK

```she
mkdir xr806_sdk 
cd xr806_sdk
git clone https://sdk.aw-ol.com/git_repo/XR806/xr806_sdk/xr806_sdk.git
```

拉取移植好的代码

microROS的编译参考[microROS的编译](XR806_microROS_uart.md)

```shell
# code 
git clone https://github.com/navy-to-haijun/microROS_XR806.git
```

文件结结构：

```shell
.
├── book		# 参考资料
│   └── Robot_Operating_System_(ROS).pdf
├── command.c
├── command.h
├── example		# 测试demo
│   ├── micro_ros_pub_int32.c		# 测试订阅
│   ├── micro_ros_pub_sub.c			# 测试发布
│   └── micro_ros_sub_int32.c		# 同时测试发布和订阅
├── gcc
│   ├── defconfig
│   └── Makefile					# 编译规则
├── image
│   └── xr806
├── main.c							# 组函数
├── microros						# 移植好的microROS
│   ├── include						# microROS头文件
│   └── libmicroros.a				# microROS静态文件库
├── prj_config.h
├── readme.md
└── transport						# 实现microROS的通信
    ├── microros_xr806.h
    ├── serial_transport.c			# uart通信实现
    └── udp_transport.c				# udp 通信实现
```



* 该demo提供microROS的静态库和相关头文件；
* 提供两种通信方式：udp和串口，通过`microros_xr806.h`中的宏定义`#define MICROROS_UDP `和`#define MICROROS_SERIAL`决定UPD相关代码是否被编译；
* 提供三个测试demo；通过`microros_xr806.h`中的宏定义`#define MICROROS_PUB_INT32`，`#define MICROROS_SUB_INT32`, `MICROROS_PUB_SUB`决定哪个demo测试被编译；

### 2 宏定义选择

```c
/*microROS相关宏定义*/
/*通信方式*/
#define MICROROS_UDP 
// #define MICROROS_SERIAL

/*demo选择*/
// #define MICROROS_PUB_INT32
// #define MICROROS_SUB_INT32
#define MICROROS_PUB_SUB
```

通信方式选择udp，demo选择MICROROS_PUB_SUB

### 3 编译

```shell
cp project/demo/ros2/gcc/defconfig .config
#检查SDK的配置是否正常
 make menuconfig
 # 清理，切换工程时需要
 make build_clean
 # 编译成镜像文件
 make build -j6
```

### 4下载

```shell
cd tools
./phoenixMC -i ../out/microros_system.img 
```

## 效果



<video src="../../picture/XR806_microROS_udp/microROS_udp_test.mp4"></video>

