# CUDA

## 1. 快速入门

https://www.cnblogs.com/skyfsm/p/9673960.html

参考项目：

https://github.com/Ewenwan/ShiYanLou/tree/master/CUDA

## 2.查看GPU的配置情况

```cpp
#include "cuda_runtime.h"
#include "device_launch_parameters.h"
#include <stdio.h>
#include <iostream>

using namespace std;
int main()
{
	int deviceCount;
	cudaGetDeviceCount(&deviceCount);

	for (int i = 0; i < deviceCount; i++)
	{
		cudaDeviceProp devPro;
		cudaGetDeviceProperties(&devPro, i);
		cout << "使用GPU：" << i << endl;
		cout << "设备全局内存总量：" << devPro.totalGlobalMem << endl;
		cout << "SM的数量:" << devPro.multiProcessorCount << endl;
		cout << "每个线程块的共享内存大小：" << devPro.sharedMemPerBlock / 1024<< "KB" << endl;
		cout << "每个线程块的最大线程数：" << devPro.maxThreadsPerBlock << endl;
		cout << "一个线程块中可用的寄存器数：" << devPro.regsPerBlock << endl;
		cout << "每个EM的最大线程数：" << devPro.maxThreadsPerMultiProcessor << endl;
		cout << "每个EM的最大线束数：" << devPro.maxThreadsPerMultiProcessor / 32<< endl;
		cout << "设备上多处理器的数量：" << devPro.multiProcessorCount << endl;
	}

	getchar();
    return 0;
}
```

## 3. CPU与GPU数据交换

```cpp
#include "cuda_runtime.h"
#include "device_launch_parameters.h"
#include <stdio.h>
#include <iostream>

using namespace std;
// 两个变量相加
__global__ void gpu_add(int a, int b, int *c)
{
	*c = a + b;
}

int main()
{
	// CPU变量
	int h_c; 

	// 指针指向GPU的内存地址
	int *d_c;

	// 分配GPU内存，存储一个int
	cudaMalloc((void**)&d_c, sizeof(int)); 

	gpu_add << <1, 1 >> > (3, 4, d_c);

	// GPU数据d_c拷贝到CPU数据h_c
	cudaMemcpy(&h_c, d_c, sizeof(int), cudaMemcpyDeviceToHost);

	printf("3 + 4 = %d\n", h_c);

	cudaFree(d_c);
	system("pause");
	return 0;
}
```

**如果输入数据在CPU上，那么需要先在GPU上分配一定的内存，再将CPU上的输入变量拷贝到GPU上进行计算。**

```cpp
#include "cuda_runtime.h"
#include "device_launch_parameters.h"
#include <stdio.h>
#include <iostream>

using namespace std;

__global__ void gpu_add(int *a, int *b, int *c)
{
	*c = *a + *b;
}

int main()
{
	int h_a, h_b, h_c; // 定义CPU上的变量
	int *d_a, *d_b, *d_c; // CPU指针变量指向GPU内存地址

	h_a = 10, h_b = 20;  // 初始化CPU变量

	// 分配GPU变量内存
	cudaMalloc((void**)&d_a, sizeof(int));
	cudaMalloc((void**)&d_b, sizeof(int));
	cudaMalloc((void**)&d_c, sizeof(int));

	// CPU上输入变量拷贝到GPU
	cudaMemcpy(d_a, &h_a, sizeof(int), cudaMemcpyHostToDevice);
	cudaMemcpy(d_b, &h_b, sizeof(int), cudaMemcpyHostToDevice);

	// 调用核函数
	gpu_add << <1, 1 >> > (d_a, d_b, d_c);

	// 拷贝GPU数据结果到CPU变量
	cudaMemcpy(&h_c, d_c, sizeof(int), cudaMemcpyDeviceToHost);

	printf("%d + %d = %d", h_a, h_b, h_c);

	// 释放GPU内存
	cudaFree(d_a);
	cudaFree(d_b);
	cudaFree(d_c);

	getchar();
	return 0;
}
```

