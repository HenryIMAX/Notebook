# Shell 基础

!!! Abstract "摘要"



​	这里主要介绍了一些Shell的基本概念和一些基础的Shell命令

​	参考资料：[竺可桢学院实用技能拾遗 Lec1](https://slides.tonycrane.cc/PracticalSkillsTutorial/2023-fall-ckc/lec1/#/)

## 1 什么是Shell？

### 1.1 Shell与Terminal

**1.1.1 什么是Terminal**

**起源:**

> - 早期计算机只有键盘，没有桌面环境
> - 用于输入数据，运行程序，并获得输出的“终端”

**现今:**

>  - Terminal Emulator，模拟传统终端的行为
>  - 一个应用程序，提供了一个窗口，和输入输出交互的功能
>  - 内部运行的是Shell，Shell才是执行命令得到输出的东西

**1.1.2 什么是Shell**

- "壳层",也是一个程序,是用户与系统内核交互的界面
- 负责接收并解析输入,交给更底层(OS)来执行,并返回输出

常见的Shell:

- Windows: cmd.exe/Powershell 5
- *nix:
  - sh:Bourne Shell(最早, 最经典)
  - bash:Bourne Again Shell(最常用)
    - 大部分Linux发行版默认的shell
  - zsh:Z Shell(功能强大, 可高度自定义)
  - fish:Friendly Interactive Shell

> 关于Shell的一些推荐配置:
>
> ![image-20250624205226903](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624205226903.png)

## 2 基础Shell命令

### 2.1 路径相关命令

- pwd：获取当前路径(print working directory)
- cd path：切换路径(chage directory)
  - path可以是“相对路径”或者“绝对路径”
  - path中 ~ 代表home，**. ** 代表当前路径，**. .**代表上一级路径

**一个例子**：

![image-20250624210413969](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624210413969.png)

## 2.2 文件/目录操作命令

![image-20250624210557776](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624210557776.png)

!!! Note "示例"

- find  **.**  -name  a.c
- find  **.**  -name  '*.c'

## 2.3 文件内容查看命令

![image-20250624230939896](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624230939896.png)

## 2.4 其他命令

![image-20250624231812612](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624231812612.png)

!!! Note "示例"

- man ls
- echo hello world

!!! tip



​	Windows是通过后缀名判断文件是否可执行(e.g. **.exe**/**.bat**/**.com**)，但是Linux和mac OS不能通过后缀名判断，需要设置可执行位，	此时需要通过chmod更改文件为可执行

## 2.5 重定向和管道

### 2.5.1 重定向

![image-20250624233741150](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624233741150.png)

### 2.5.2 管道

- 通过管道(pipe)可以将一个命令的输出作为另一个命令的输入
- 使用管道操作符 | ，将左侧命令的stdout连接到右侧命令的stdin
- 通过管道可以将多个命令连接起来，形成一个命令序列，可以通过一行命令来完成相对复杂的动作

 !!! Note "常见用法"

- some cmd | tail -n lines：只输出最后lines行

  e.g. ls | head -n 10 | tail -n 5：输出第5~10个文件名

- some cmd | less：分页输出
- some cmd | grep pattern：在输出中查找匹配pattern的行
- 与cut / sort / uniq / awk等命令搭配，处理文本数据
- . . .

## 2.6 环境变量

![image-20250624235735289](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250624235735289.png)

!!! Note "示例"

- export GOOD_MORNING_MY_NEIGHBORS = "hey"

- unset GOOD_MORNING_MY_MEIGHBORS