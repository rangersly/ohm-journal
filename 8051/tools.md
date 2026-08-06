# 8051开发工具链

- [wsl连接到物理机](#usbipd)
- [编译工具](#sdcc)
- [烧录工具](#stcgal)
- [内存分析](#ram)

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
|`sfr P0 = 0x80;`|`__sfr __at(0x80) P0;`|
|`sbit P0_0 = 0x80;`|`__sbit __at(0x80) P0_0;`|
|`data`,`idata`,`xdata`,`code`|`__data`,`__idata`,`__xdata`,`__code`|
|`interrupt`|`__interrupt(n)`|

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
|`-o <option>`|设置参数,例如6t模式|
|`-b <波特率>`|指定波特率(默认115200)|

`stcgal -p <设备>` 这个方法可以检测芯片信息

## ram

8051的内存吃紧,需要分析编译完成后的 `.map` 文件以保证不会出错

|区域|作用|
|---|---|
|`CABS`|`falsh` 存储大小|
|`SEG`|RAM大小,一般看`SSEG`栈顶指针位置|
|`CONST`|常量|

又研究了一下,感觉 `.mem` 文件更好看,但是有几个坑,下面贴出 `.mem` 内容解释

```
Internal RAM layout:
      0 1 2 3 4 5 6 7 8 9 A B C D E F
0x00:|0|0|0|0|0|0|0|0|a|a|a|a|a|a|a|a|
0x10:|Q|Q|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x20:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x30:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x40:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x50:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x60:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x70:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x80:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0x90:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0xa0:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0xb0:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0xc0:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0xd0:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0xe0:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0xf0:|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|S|
0-3:Reg Banks, T:Bit regs, a-z:Data, B:Bits, Q:Overlay, I:iData, S:Stack, A:Absolute

Stack starts at: 0x12 (sp set to 0x11) with 238 bytes available.
No spare internal RAM space left.

Other memory:
   Name             Start    End      Size     Max     
   ---------------- -------- -------- -------- --------
   PAGED EXT. RAM                         0      256   
   EXTERNAL RAM                           0    65536   
   ROM/EPROM/FLASH  0x0000   0x0186     391    65536   
```

|标记|作用|
|---|---|
|`0`|0x00~0x07八字节为工作寄存器|
|`a(DATA)`|0x08~0x0F,全局/静态变量存放区|
|`Q(Overlay)`|覆盖层区域,将互不调用的函数中局部变量或参数复用在这里|
|`S`|栈区,低于0x7F为硬件栈,SP指针可以到的地方,保存返回地址和中断现场,而高区只能用来存放临时变量等|

> [!IMPORTANT]
> 0x20~0x2F为可位寻址区

> [!NOTE]
> 可以通过 `.lst` 来找PUSH指令分析栈顶极限位置
