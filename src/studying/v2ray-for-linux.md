# v2ray for linux

## 下载

```

https://github.com/v2ray/v2ray-core

https://github.com/v2fly/v2ray-core/releases/tag/v4.31.0

# 选择对应版本
```

## 配置

将`v2ray`移植到`/opt/`下

保证v2ray可以正常运行

```shell
# 检查是否可以运行
./v2ray help
# 查询运行命令
./v2ray help run
# 执行
/opt/v2ray/v2ray run -c /opt/v2ray/config.json
```

建立守护进程

`sudo vim /etc/systemd/system/v2ray.service`

```shell
[Unit]
Description=v2ray
[Service]
Type=simple
ExecStart=/opt/v2ray/v2ray run -c /opt/v2ray/config.json
[Install]
WantedBy=multi-user.target
Alias=v2ray.service
```

命令

```shell
export all_proxy="socks://127.0.0.1:10808/"
export http_proxy="http://127.0.0.1:10809/"
export https_proxy="http://127.0.0.1:10809/"

# 启动V2ray
sudo systemctl start v2ray

# 检查V2ray状态
sudo systemctl status v2ray

# 设置V2ray开机自启动
sudo systemctl enable v2ray
```

测试

```shell
curl -i google.com
```

网络设置

系统设置

`设置->网络`

![网络设置(系统)](../picture/v2ray-for-linux/网络设置(系统).png)