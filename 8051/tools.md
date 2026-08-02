# 8051开发工具链

- [wsl连接到物理机](#usbipd)
- [编译工具](#sdcc)
- [烧录工具](#stcgal)

## usbipd

对于使用wsl2进行开发的,需要使用usbipd将连接转发到wsl2中

`winget install usbipd` 下载usbipd,如果不行,到github下载`.msi`文件

启动一个wsl2终端保持运行

`usbipd list` 记录下单片机设备的 `BUSID`

`usbipd bind --busid <busid>` 绑定设备

`usbipd attach --wsl --busid <BUSID>` 连接到设备

最后在wsl2环境中验证连接即可 `ls /dev/tty*`

如果遇到权限问题记得手动给`ttyUSB`赋予666权限

## sdcc

`sdcc <filename.c>`

- **输出文件**：
  - `main.ihx`：Intel Hex 格式的固件文件（可直接烧录）。
  - `main.asm`：生成的汇编代码（便于调试）。
  - `main.lk`：链接器脚本。
  - `main.map`：内存映射和符号表。
  - `main.rel` : 参与连接的文件

### **特殊语法**

针对8051单片机

- **内存类型修饰符**
- `__code` : 存储在 Flash, 通常为常量,字符串
- `__data`：将变量存储在内部 RAM（直接寻址区）。
- `__idata`：将变量存储在内部 RAM（间接寻址区）。
- `__xdata`：将变量存储在外部 RAM（扩展 XDATA 区）。
- `sfr/sbit`：特殊寄存器
- `__bit`：定义位变量（仅限 8051 的位寻址区）。

### 指针

sdcc 支持两种指针:

|类型|效果|
|---|---|
|通用指针(3B)|可指向任意空间,灵活但体积大,速度慢|
|指定存储空间的指针(1~2B)|如`unsigned char xdata *p;`仅指向xdata,但效率高|

### **优化选项**

- **常用编译选项**：
  - `--model-<model>`：指定内存模型（如 `--model-small`、`--model-large`使所有未指定变量默认分配到外部RAM）。
  - `--opt-code-size`：优化代码大小。
  - `--nogcse` : 按需选择,避免未使用的 函数/变量 占用空间
  - `--stack-auto`：自动分配堆栈（适用于函数调用）。
  - `--nooverlay`：禁用函数参数和局部变量的覆盖优化。
  - `--verbose`：显示详细编译过程。

### 与KEIL的写法差异

|KEIL|SDCC|
|---|---|
|`sfr P0 = 0x80;`|`__sfr at(0x80) P0;`|

## stcgal

### 准备工作

采取python虚拟环境的方式进行部署

确保安装了`python`和`pip`

`python -m venv .venv` 创建一个虚拟环境(在工作目录下运行)

安装`direnv` 用于自动激活虚拟环境

```
# 在指定项目下输入虚拟环境指令
cd ~/my_stc_project
echo "source .venv/bin/activate" > .envrc
# 授权
direnv allow
```

`pip install stcgal` 安装烧录程序

### 核心用法

`stcgal [选项] [代码文件] [可选EEPROM文件]`

### 关键参数

|参数|功能|
|---|---|
|`-p <串口>`|指定串口设备|
|`-b <波特率>`|指定波特率(默认115200)|

`stcgal -p <设备>` 这个方法可以检测芯片信息
