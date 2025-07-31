# OpenMP & MPI 

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702084059072.png" alt="image-20250702084059072" style="zoom:50%;" />

## OpenMP 

**定义：**Open Multi-Processing，支持**C，C++，Fortran**的一种API，和编译器强相关

**OpenMP provides us an easy way to transform serial programs into parallel.**

### **Example 1: Parallel for hello world**

~~~ C
#include <stdio.h>
#include <omp.h>		//添加OpenMP头文件
int main(){
    printf("Welcome to OpenMP!\n");
    #pragma omp parallel		//Preprocessing Directive
    {
        int ID = omp_get_thread_num();		 //Parallel Region
        printf("hello(%d)", ID);			 
        printf("World(%d)\n", ID);			 
    }										 
    printf("Bye\n");
}
~~~

### Fork-Join Model

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702085313721.png" alt="image-20250702085313721" style="zoom:50%;" />

**图中的分支点称为Fork，合并点称为Join**

抽象原理图：

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702085349059.png" alt="image-20250702085349059" style="zoom:50%;" />

**这里的TID=0的线程就是主线程**

### OpenMP directive

[官方文档](https://www.openmp.org/wp-content/uploads/OpenMPRefCard-5-2-web.pdf)

### Constructs

**An OpenMP Constructs is a formation for which the directive is executable.**

Work-distribution Constructs:

- single
- **section**
- for

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702090339592.png" alt="image-20250702090339592" style="zoom:50%;" />

**Section:**

~~~ c
#pragma omp parallel ...
{
    {
        SECTION 1
    }
    {
        SECTION 2
    }
    {
        SECTION 3
    }
}
~~~

每个Section中的内容可以不一样

编译指令示例：

```bash
gcc a.c -o a -fopenmp
```

### **Example 2：parallel for Directive**

``` C
#pragma omp parallel for
//没有{}，将for循环内部分配到不同线程
for(int i=0; i<N; i++){
    c[i] = a[i] + b[i];
}

//nproc指令可以查看CPU可以同时处理多少个线程
```

实际上并行速度提升(Speed Up)和CPU核心数(可同时进行的线程数)并非线性关系，这是因为**overhead**：

- 创建和销毁线程
- 操作系统调度
- OpenMP本身的调度
- NUMA造成的内存访问时延不一致等等

### **Example 3: Loop Schedule**

```C
#pragma omp parallel for
for(int i=0;i<N;i++){
    c[i] = f(i);      //Workload is unbalanced!此时需要for后面的clause修饰
}
```

- Static 调度：执行前完成调度，减少执行时OpenMP的调度

- Dynamic调度：根据任务数量动态调整并行度，但是OpenMP维护开销大

  （schedule(dynamic ,1)，其中1表示调度的粒度）

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702093412116.png" alt="image-20250702093412116" style="zoom:50%;" />

### **Example 4：Nested for loop**

```C
//Matrix Element-wise Addition
#pragma omp parallel for collapse(2)
for(int i=0;i<N;i++){
    for(int j=0;j<N:j++){
        c[i][j] = a[i][j] + b[i][j];
    }
}
```

### **Shared Data and Data Hazards**

**data hazards**:不同核心执行过程中不知道另一个线程进行到什么程度，由于数据共享多个线程会访问同一变量，导致data hazards，如下图

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702094918668.png" alt="image-20250702094918668" style="zoom:50%;" />

```C
#pragma omp parallel for
for (int i = 0; i < 100; i++) {
	sum += a[i];
}
```

**Solution：**

- private：每个线程有自己的变量，互不干扰

``` C
#pragma omp parallel for private
for (int i = 0; i < 100; i++) {
	sum += a[i];
}
```

- shared

- Critical Section

  ```C
  #pragma omp parallel for
  	for (int i = 0; i < 100; i++) {
  #pragma omp critical
  		{ sum += a[i]; }//只允许一个线程进入，相当于此部分串行
  	}
  	printf("Sum = %d\n", sum)
  ```

- Atomic Operation

  ```C
  #pragma omp parallel for
  	for (int i = 0; i < 100; i++) {
  #pragma omp atomic     //原子操作，不可分割，保证多个线程对一个变量的操作是原子的
          //不会分成多个操作，使用CPU原子化指令，只适用于简单操作（四则运算和位运算等）
  		sum += a[i];
  	}
  	printf("Sum = %d\n", sum);
  ```

- Reduction（推荐）

  ```C
  #pragma omp parallel for reduction(+:sum)	//+是操作，sum是变量
  for (int i = 0; i < 100; i++) {
  	sum += a[i];
  }
  printf("Sum = %d\n", sum);
  ```

**Comparison**

- Critical Region: Based on locking

- Atomic Operation: Based on hardware atomic operations

- Reduction: only synchronize in the end

### Example 5: GEMM

```C
//General Matrix Multiplication (GEMM)
#pragma omp parallel for collapse(3) reduction(+ : c)
	for (int i = 0; i < N; i++) {
		for (int j = 0; j < N; j++) {
		c[i][j] = 0;
			for (int k = 0; k < N; k++) {
				c[i][j] += a[i][k] * b[k][j];
			}
		}
	}
```

### False Sharing

一个核心改变了cache中的内容后另一个核心的cache就失效了，需要重新读取，所以在多线程访问时需要让每一段core分割的大小大于cache line

### Summary

**How to Optimize a program with OpenMP:**

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702101229564.png" alt="image-20250702101229564" style="zoom:50%;" />

## MPI 

- Intel-MPI
- OpenMPI
- MPICH
- HMPI

```C
#include <mpi.h>
#include <stdio.h>
    int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);
    printf("Hello world from processor %s, rank %d out of %d processors\n",
    processor_name, world_rank, world_size);
    MPI_Finalize();		//不要重复的init,finalize,做一次就行
    return 0;
}
```

### Basic Concepts

- rank:每个进程的编号

- MPI_COMM_SPLIT：将进程分组

- Blocking：通信结束后再进行后续操作

  Non-Blocking：通信的同时就可以进行后续操作

- Order：保证数据包传输和接收的顺序是一致的，不会颠倒，但不会保证数据传输的公平性

  <img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702103225174.png" alt="image-20250702103225174" style="zoom:50%;" />

  有可能Rank1一直占用导致Rank2无法进行传输

### Point-to-Point Communication

MPI send函数和MPI receive函数

```C
int MPI_Send(
    const void* buffer,		//数据内存起始指针，指向要传输的数据
    int count,				//传输的数量
    MPI_Datatype datatype,	//数据的类型
    int recipient,			//目标进程ID
    int tag,				//消息标签，一般为0
    MPI_Comm communicator	//通信域，一般为MPI_COMM_WORLD
);

int MPI_Recv(
    void* buffer,
    int count,
    MPI_Datatype datatype,
    int sender,				//发送进程ID
    int tag,
    MPI_Comm communicator,
    MPI_Status* status
    //MPI_Status represents the status of a reception operation
    //一般有三种：MPI_SOURCE，MPI_TAG，MPI_ERROR
    
);  //阻塞函数
```

**A common mistake**

```C
//n = 2
MPI_Comm_rank(comm, &my_rank);
MPI_Ssend(sendbuf, count, MPI_INT, my_rank ^ 1, tag, comm);
MPI_Recv(recvbuf, count, MPI_INT, my_rank ^ 1, tag, comm, &status);
```

MPI_Ssend()函数是阻塞函数，需要等到对方接收后才会继续执行

:warning:上面的程序会发生deadlock

**Solution**

```C
//n = 2
MPI_Comm_rank(comm, &my_rank);
if (my_rank == 0) {
    MPI_Ssend(sendbuf, count, MPI_INT, 1, tag, comm);
    MPI_Recv(recvbuf, count, MPI_INT, 1, tag, comm, &status);
} else if (my_rank == 1) {
    MPI_Recv(recvbuf, count, MPI_INT, 0, tag, comm, &status);
    MPI_Ssend(sendbuf, count, MPI_INT, 0, tag, comm);
}
```

Or Use MPI_Sendrecv

```C
int MPI_Sendrecv(
    const void* buffer_send,		//Notice:The buffers used for send and receive must be different.
    int count_send,
    MPI_Datatype datatype_send,
    int recipient,
    int tag_send,
    void* buffer_recv,
    int count_recv,
    MPI_Datatype datatype_recv,
    int sender,
    int tag_recv,
    MPI_Comm communicator,
    MPI_Status* status
);
```

Or Use Non-Blocking Function(不等对方接收到就继续执行)

```C
int MPI_Isend(
    const void* buffer,
    int count,
    MPI_Datatype datatype,
    int recipient,
    int tag,
    MPI_Comm communicator,
    MPI_Request* request
);
int MPI_IRecv(...);
```

如何确定对方确实接收到数据：

- MPI_Test：非阻塞，检查通信是否完成，只是看一眼，仍然继续运行
- MPI_Wait：阻塞，等待通信完成后再继续

应用：先TEST，中间执行一些不需要数据依赖的计算，需要依赖时再Wait

### Collective Communication

- **Broadcast: one to all**

  ```C
  int MPI_Bcast(
      void* buffer,
      int count,
      MPI_Datatype datatype,
      int emitter_rank,		//广播发起者，其他进程需要等该主进程通信之后在执行
      MPI_Comm communicator
  );
  ```

  **优点：提升传输效率，传输到一个进程之后该进程可以继续向其他进程发送($logN$)**

- **Synchronization**

  ```C
  MPI_Barrier(COMM)   //如果某一进程调用了此函数，需要等待其他进程到达此barrier后再继续
  ```

  <img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702105643859.png" alt="image-20250702105643859" style="zoom:50%;" />

- **Scatter(One to All)**

```C
int MPI_Scatter(
    const void* buffer_send,
    int count_send,				//The number of elements to send to each process
    MPI_Datatype datatype_send,
    void* buffer_recv,
    int count_recv,				//The number of elements in the receive buffer
    MPI_Datatype datatype_recv,
    int root,
    MPI_Comm communicator
);
```

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702110029762.png" alt="image-20250702110029762" style="zoom:40%;" />

- **Gather(All to One)**

```C
int MPI_Gather(
    const void* buffer_send,
    int count_send,
    MPI_Datatype datatype_send,
    void* buffer_recv,
    int count_recv,
    MPI_Datatype datatype_recv,
    int root,
    MPI_Comm communicator
);
```

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702110249771.png" alt="image-20250702110249771" style="zoom:40%;" />

**Example:Compute Average**

```C
MPI_Scatter(buffer, 0x1000000/4, MPI_DOUBLE, local_buffer, 0x1000000/4, MPI_DOUBLE, 0, MPI_COMM_WORLD);
double local_avg = 0;
for(int i=0; i<0x1000000/4; i++){
	local_avg += local_buffer[i];
}
local_avg /= 0x1000000/4;
double avgs[4];
MPI_Gather(&local_avg, 1, MPI_DOUBLE, avgs, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
```

- **AllGather(All to All)**

  ```C
  int MPI_Allgather(
      const void* buffer_send,
      int count_send,
      MPI_Datatype datatype_send,
      void* buffer_recv,
      int count_recv,
      MPI_Datatype datatype_recv,
      MPI_Comm communicator);
  ```

  可以理解为先Gather到主进程0之后，再进行Broadcast操作

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702110433144.png" alt="image-20250702110433144" style="zoom:40%;" />

- **Reduce**

  ```C
  int MPI_Reduce(
      const void* send_buffer,
      void* receive_buffer,
      int count,
      MPI_Datatype datatype,
      MPI_Op operation,
      int root,
      MPI_Comm communicator
  );
  ```

  <img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702110603625.png" alt="image-20250702110603625" style="zoom:40%;" />

**Example:Compute average revisit**

```C
MPI_Scatter(buffer, 0x1000000/4, MPI_DOUBLE, local_buffer, 0x1000000/4, MPI_DOUBLE, 0, MPI_COMM_WORLD);
double local_avg = 0;
for(int i=0; i<0x1000000/4; i++){
	local_avg += local_buffer[i];
}
local_avg /= 0x1000000/4;
double global_avg;
MPI_Reduce(&local_avg, &global_avg, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
```

### **Miscellaneous**

**OpenMPI编译使用**

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250702112334128.png" alt="image-20250702112334128" style="zoom:50%;" />

**mpirun的用法**

- -x[env]: 指定环境变量
- -bind -to core: 绑核
- -hostfile [hostfile]