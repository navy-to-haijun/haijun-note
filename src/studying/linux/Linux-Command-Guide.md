# linux 常用命令 

#### cp

```she
cp -rfd dir1 dir2
```

将dir1文件复制到dir2中

* `r`：recursive，递归地，即复制所有文件 ;
* `f`:force，强制覆盖;
* `d`:如果源文件为链接文件，也只是把它作为链接文件复制过去，而不是复制实际文件 ;

#### 文件权限

![image-20230718172815430](../../picture/Linux-Commend-Guide/image-20230718172815430.png)

- r: 读：4
- w：写：2
- x：执行：1



* owner: rwx;
* group: r-x;
* others: r-x;

改变权限

将owner的权限改变为只有可执行，其他不变

```shell
chmod 155 exec
#or
chmod u-w-r exec
```

* u、g、o三个字母代表 user、group、others 3中身份。此外a代表all，即所有身份。 
* `+`: 增加权限；
* `-`：去除权限；

#### 查找/搜索命令

`find`：查找文件

```shell
find 目录名 选项 查找条件
# 查找当前文件夹下的test.txt文件
find ./ -name "test.txt"
```

`grep`:是查找文件中符合条件的字符串

```shell
grep [选项] [查找模式] [文件名]
# 搜索当前文件夹下含有`abc`的字符串，并显示行号
grep -rnw 'abc' ./
```

* `r`：递归；
* `n`：所在行号；
* `w`: 全字匹配；

#### 解压缩

`tar`

* `-c` (create)：表示创建用来生成文件包 。 
* `-x`表示提取，从文件包中提取文件。
* `-t`可以查看压缩的文件。
* `-z`使用 gzip 方式进行处理，它与”c“结合就表示压缩，与”x“结合就表示解压缩。 
* `-j` -j：使用bzip2方式进行处理，它与”c“结合就表示压缩，与”x“结合就表示解压缩。 
* `-v`(verbose)：详细报告 tar处理的信息
* `-f`(file)：表示文件，后面接着一个文件名。 -C <指定目录> 解压到指定目录。

压缩

```shell
tar -czf makefile.tar.gz makefile
tar -cjf makefile.tar.bz2 makefile
```

解压

```shell
tar -xzf makefile.tar.gz -C ../test1
tar -xjf makefile.tar.gz -C ../test2
```

查看

```shell
tar -tzf makefile.tar.gz
tar -tjf makefile.tar.bz2
```



