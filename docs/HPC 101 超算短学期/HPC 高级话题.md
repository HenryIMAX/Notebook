# HPC 高级话题

**HPC System:Four  Components**

- Computing
- Storage
- Networking
- Compiling

## Conputing

**人工智能大模型算力估计**

- $数据量(D)>15\times模型参数量(N)$
  - $万亿模型(N) = 1000*10^9 = 10^{12}$
  - $数据量(D) > 15*10^{12} = 1.5*10^{13}$
- $计算次数C \approx 6\times N \times D$
  - $万亿模型计算次数C \approx6*N*D \approx9*10^{25}$

**CPU:Central Processing Unit(一个大学生)**

**GPU:Grarphics Processing Unit(100个小学生)**——适合同时执行多个简单操作

**FPGA:Field Programable Gate Array(蓝翔技校)**——适合做定制化计算

性能比较：

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703150531716.png" alt="image-20250703150531716" style="zoom:50%;" />

### CPU

Intel CPU架构

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703150917205.png" alt="image-20250703150917205" style="zoom:50%;" />

CPU经典5级流水线：

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703152158463.png" alt="image-20250703152158463" style="zoom:50%;" />

- 优点：一条指令操作一个数，灵活，可实现任意功能函数。
- 劣势：效率很低，五个流水线模块只有EXE模块在真正做计算

CPU并行方式SIMD：一个周期完成多个加法

CPU：超标量Superscalar

- CISC指令内部RISC化——读入CISC指令，转换成RISC后再执行
- 指令多并发：4条微指令并发，6条CISC指令一起解析

### GPU

Nvidia GPU 架构：

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703162050651.png" alt="image-20250703162050651" style="zoom:50%;" />

矩阵指令：减少周期数，每个周期内存访问量增大，充分利用内存

示例：矩阵乘法 `C[:] [:] = A[:] [:] + B[:] [:]`

GPU上使用Tensor Cores的编程：

~~~c++
#include<mma.h>
using namespace nvcuda;
__global__ void wmma_ker(half *a, half *b, float *c){
    //Declare the fragments
    wmma::fragment<wmma::matrix-a,16,16,16,half,wmma::col_major> a_frag;
    wmma::fragment<wmma::matrix-b,16,16,16,half,wmma::col_major> b_frag;
    wmma::fragment<wmma::accumulator,16,16,16,float> c_frag;
    
    //Initialize the output c_frag to zero
    wmma::fill_fragment(c_frag, 0.0f);
    
    //Load the inputs a_frag,b_frag
    wmma::load_matrix_sync(a_frag,a,16);
    wmma::load_matrix_sync(b_frag,b,16);
    
    //Perform matrix multiplication c_frag = a_frag * b_frag + c_frag
    wmma::mma_sync(c_frag,a_frag,b_frag,c_frag);
    
    //Store the output c_frag
    wmma::store_matrix_sync(c,c_frag,16,wmma::mem_row_major);
}
~~~

GPU的限制：PCIe的带宽(I/O)

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703163940210.png" alt="image-20250703163940210" style="zoom:50%;" />

PCIe带宽:64GB/s  GPU带宽：3000GB/s

### FPGA

特点：可实现位级别的并行度，是独立的系统，是Memory，Network，CPU，GPU的粘合剂

**FPGA编程**：并行度高，组合逻辑，时序逻辑

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703164645677.png" alt="image-20250703164645677" style="zoom:50%;" />

## Networking

- Scale Up Network
- Scale Out Network

具体知识可参考计算机网络

## Storage

几类存储：

- Flip-Flop：速度快，并行度高，成本很贵
- SRAM：相对较快，并行度不太高，每次处理一个data word，成本贵(几MB)
- DRAM：速度慢，但是容量大（几GB），成本便宜，但是使用有损耗
- Flash Memory(闪存)：非常慢，十分便宜，容量很高

性能对比：

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250703170755586.png" alt="image-20250703170755586" style="zoom:50%;" />

——  **latency:延迟  capacity:容量  bandwidth:带宽**

## AI计算系统：异构+分布式

