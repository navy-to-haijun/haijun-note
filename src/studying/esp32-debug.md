# esp32-debug



`.vscode/launch.json`配置文件

example

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "espidf",
      "name": "Launch",
      "request": "launch",
      "debugPort": 9998,
      "logLevel": 2,
      "mode": "manual",
      "verifyAppBinBeforeDebug": false,
      "tmoScaleFactor": 1,
      "initGdbCommands": [
        "target remote :3333",
        "symbol-file /path/to/program.elf",
        "mon reset halt",
        "flushregs",
        "thb app_main"
      ],
      "env": {
        "CUSTOM_ENV_VAR": "SOME_VALUE"
      }
    }
  ]
}
```

1. `type`：只能是`espidf`;
2. `name`：调试名称，不重要
3. `request`
4. `debugPort`：调试器的端口，默认43474
5. `logLevel`：日志级别，默认为2
   1. `mode`参数为`auto`和`manual`，`auto`自动启动openOCD服务器，`manual`手动启动
6. `verifyAppBinBeforeDebug`：默认为`false`，调试只能调用工程下的可执行文件，改为`ture`，可以手动指定可执行文件
7. `tmoScaleFactor`：超时时间
8. `initGdbCommands`
9. `env`：设置环境变量