# CircuitPython

## 参考连接

https://docs.circuitpython.org/en/latest/README.html

## build

https://learn.adafruit.com/building-circuitpython/build-circuitpython

conda 创建新环境

```bash
conda create --name esp32 python=3
# 查看虚拟和环境
conda env list 

conda activate esp32

conda deactivate
```

安装依赖

```bash
pip3 install --upgrade -r requirements-dev.txt
pip3 install --upgrade -r requirements-doc.txt
```

修改分支

```bash
git checkout 8.2.x
```

获取子模块

```bash
# 获取全部子模块
make fetch-all-submodules
# 获取指定子模块
make fetch-port-submodules
# 删除全部子模块
make remove-all-submodules

```

## Build mpy-cross

将Circuitpython **.py** files 编译为.mpy文件

```bash
make -C mpy-cross
```

编译 circuitpython

编译相应的板子

```bash
cd ports/atmel-samd
make BOARD=circuitplayground_express TRANSLATION=es
```

# displayio

位置关系

![circuitpython_coord_sys.png](https://cdn-learn.adafruit.com/assets/assets/000/074/495/medium800/circuitpython_coord_sys.png?1555378384)
