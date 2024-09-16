# esp32-usb

​	ESP32-S3 带有一个集成了收发器的 USB On-The-Go（下文将称为 OTG_FS）外设。完全符合 USB 2.0 协议规范。它支持传输速率为 12 Mbit/s 的全速模式 (Full-Speed, FS) 和传输速率为 1.5 Mbit/s 的低速模式 (Low-Speed, LS)

当仅使用集成收发器时，可通过时分复用技术，和 USB Serial/JTAG 控制器共用集成收发器

设备模式 (Device mode) 特性：

* 端点 0 永远存在（双向控制，由 EP0 IN 和 EP0 OUT 组成
* 6 个附加端点 (1 ~ 6)，可配置为 IN 或 OUT
* 最多 5 个 IN 端点同时工作（包括 EP0 IN）
* 所有 OUT 端点共享一个 RX FIFO
* 每个 IN 端点都有专用的 TX FIFO

## USB 设备栈

基于 TinyUSB 栈构建，但对 TinyUSB 进行了一些小的功能扩展和修改，使其更好地集成到 ESP-IDF