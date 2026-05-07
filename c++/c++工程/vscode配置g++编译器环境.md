#### 1.安装环境管理工具MSYS2

打开地址[Releases · msys2/msys2-installer](https://github.com/msys2/msys2-installer/releases/)，下载msys2-x86_64-20260322.exe并安装

------

#### 2.利用MSYS2安装编译器环境MinGW-w64

打开**MSYS2 UCRT64终端**

![image-20260507153508797](../../图片/image-20260507153508797.png)

输入命令

```shell
pacman -S --needed base-devel mingw-w64-ucrt-x86_64-toolchain
```

------

#### 3.配置环境变量

将**C:\msys64\ucrt64\bin**放进系统变量的Path里即可

------

#### 4.验证环境

```cmd
gcc --version
g++ --version
gdb --version
```

