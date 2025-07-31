# Chapter 2 Representing and Manipulating Information

!!! Abstract "本章概要"

## 引言

本章我们主要考虑三种数字的表示方式：

- *Unsigned* Encoding:用来表示非负数
- *Two's-Complement* Encoding：用来表示有符号整数
- *Floating-point* Encoding：用来表示实数

对于整数运算，虽然有可能出现***overflow***，但是满足正常运算中的运算律，例如
$$
(500 \times  400) \times (300 \times 200) =((500 \times 400)\times300)\times200 = 400\times(200\times(300\times500))
$$
但是对于浮点数运算和正常的运算律不同，例如
$$
(3.14+1e20)-1e20 \ne 3.14+(1e20-1e20)
$$
前者得到的结果是0.0，而后者得到的结果是3.14

## 2.1 Information Storage

在计算机中，信息以**字节(bytes)**为单位 (可被处理的最小内存单元，注意==不是bit==)存储在内存当中。计算机程序将内存时为一个非常大的字节数组，称为**虚拟内存(virtual memory)**，内存中的每个字节通过一个特定的数字来标识，称为**地址(address)**，所有可能的地址的集合称为**虚拟地址空间(virtual address space)**。

在之后的章节中，我们会了解编译器和运行中的程序是如何将内存空间分成可管理的单元来储存不同的**程序对象(program objects)**，即程序数据，指令和控制信息。以C语言的指针为例，指针的值就是程序对象第一个字节的虚拟地址，通过类型(type)确定如何读取其中的值。

### 2.1.1 Data Sizes

计算机和编译器通过使用不同方法来编码数据，例如整数和浮点数，它们都有不同的长度。例如下图中展示了C语言中不同数据类型的字节数：

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250630220416503.png" alt="image-20250630220416503" style="zoom: 67%;" />

每个数据类型的字节数取决于机器和编译器，上图中展示的就是32位和64位两种不同机器中各个数据类型的字节数，long数据类型占据了该机器的整个**字长(word size)**，在32位中就是4个字节，64位中就是8个字节。long long数据类型(ISO C99)允许64位的整数，在332位的机器中，编译器必须通过生成执行32位操作序列的代码来为这种数据类型编译运算操作。C语言中的指针也占据了机器的整个字长。浮点数分为单精度(float)和双精度(double)，分别为4个字节和8个字节。

### 2.1.2 Addressing and Byte Ordering

