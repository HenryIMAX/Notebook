# CUDA C 编程基础

## 异构计算(Heterogeneous Computing)

- Host：CPU及其内存
- Device：GPU及其内存

CPU和GPU通过PCI总线进行通信，在GPU上进行计算时先将CPU内存中的数据送到GPU内存中，运算之后再将结果送回CPU内存

编译器nvcc，文件后缀.cu

Hello World

~~~ c
__global__ void mykernel(){} //global 可以在CPU和GPU上运行
int main(){
    mykernel<<<1,1>>>(); //调用GPU上的kernel
    pritf("Hello World from GPU\n");
}
~~~

~~~ C
__global__ void add(int *a, int *b, int *c){
     *c = *a + *b;   
}

int main(){
    int a,b,c;				//host copies of a,b,c
    int *d_a, *d_b, *d_c;	//device copies of a,b,c
    int size = sizeof(int);
    //Allocate space for device copies
    cudaMalloc((void **)&d_a, size);
    cudaMalloc((void **)&d_b, size);
    cudaMalloc((void **)&d_c, size);
    //copy inputs to device
    cudeMemcpy(d_a, &a, size, cudaMemcpyHostToDevice);
    cudeMemcpy(d_b, &b, size, cudaMemcpyHostToDevice);
    //Launch add() kernel on GPU
    add<<<1,1>>>(d_a,d_b,d_c);
    //copy result back to host
    cudaMemcpy(&c, d_c, size,cudamemcpyDeviceToHost);
    //cleanup
    cudaFree(d_a);cudaFree(d_b);
    cudaFree(d_c);
    return 0;
}

add<<<N,1>>>(d_a,d_b,d_c) //N 表示并行执行次数，启动了N个block
add<<<1,N>>>(d_a,d_b,d_c) //N表示一个block中的线程个数
~~~

在GPU上分配内存：

## Block and Thread

**定义 ：**`add()`的每次并行调用称为一个block

- block的集合称为grid
- 每次调用都可以使用`blockIdx.x`来引用其block index

```C
__global__ void add(int *a, int *b, int *c){
    c[blockIdx.x] = a[blockIdx.x] + b[blockIdx.x];
}
```

通过`blockIdx.x`使得每个block处理不同的index

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20250701145314413.png" alt="image-20250701145314413" style="zoom:50%;" />



**Shared Memory:**在同一块block中，线程可以互相share data

**卷积：**

~~~ C
__global__ void stencil_1d(int *in, int *out){
    __shared__ int temp[BLOCK_SIZE + 2 * RADIUS];
    int gindex = threadIdx.x + blockIdx.x * blockDim.x;
    int lindex = threadIdx.x + RADIUS;
}
~~~

