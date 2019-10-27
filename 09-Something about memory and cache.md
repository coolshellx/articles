### Cache与内存二三事
本文搜集了几个与内存和缓存相关的技巧，对于代码调优比较有帮助。通过下文我们可以知道，看似对软件工程师透明的内存以及CPU Cache，其实并不“透明”，代码的细微差别可以明显的影响缓存以及内存的性能。

### 1.会导致缺页中断的内存分配
下文代码采用两种方式分配pBuffer，请比对两种不同方式的耗时。

```c++
#include "stdafx.h"
#include <iostream>
#include <chrono>
#include <math.h>
using namespace std::chrono;

const int buff_size = 32 * 1024 * 1024;

int main()
{
	int * pBuffer = new int[buff_size]; //第一种分配方式
	//int * pBuffer = new int[buff_size] {}; //第二种分配方式

	auto start = std::chrono::system_clock::now();
	for (int i = 0; i < buff_size; ++i) {
		pBuffer[i] *= 3;
	}
	auto end = std::chrono::system_clock::now();

	std::chrono::duration<double> elapsed_seconds = end - start;
	std::cout << "elapsed time: " << elapsed_seconds.count() * 1000 << "ms\n";

	delete[] pBuffer;
}
```

在我的电脑上，第一种分配方式的运行时间大约为90ms，第二种分配方式的运行时间大约为26ms。两种分配方式唯一的区别在于后者在分配内存的时候添加了 {}。
当我们从堆里分配一个数组的时候，如果没有做任何初始化，那么操作系统只会返回一个虚拟地址映射，并不会真正地分配一块物理内存。当这个数组被真正访问的时候，一个缺页中断会被触发，内核才会真正分配一块物理内存。第一种分配方式在分配内存的时候，没有做初始化，所以访问内存的时候会导致缺页中断；第二种分配方式添加了{}强制初始化，因此在分配数组的时候内核就分配了物理内存。在本例中，缺页中断比较明显地影响了程序的性能。

### 2. 如何测量cpu cache的容量
下面这段代码可以在linux上测量缓存的容量。基本思路是，step1:以某个步长逐一访问一块内存的存储单元，然后统计访问时间。step2:改变步长，重复step1。step3:最后比较相邻两次的访问时间，找到差别最大的一次，这个步长就是cache容量。这个方法的原理是，当缓存命中的时候即使步长不相等，访问内存的时间是差不多的；当缓存没有命中的时候，因为需要重新导入内存到缓存，会导致内存访问时间大大增加。所以，若要充分利用缓存，我们最好以地址连续的方式访问内存，尽量避免缓存的内容抖动。

以下代码来自https://github.com/SudarsunKannan/memlatency

```c++
void LLCCacheSizeTest(unsigned int arg)
{
	double retval,prev;
	struct timespec tps, tpe;
	int max =INT_MIN,index = 0, maxidx = 0, iter = 0, i=0;
	int diff =0, stride = 0, j = MIN_LLC;
	double change_per[10];
	char *array = (char *)malloc(sizeof(char) *CACHE_LN_SZ * STEPS_N);

	while( j < STOP_SIZE) {
		stride = j-1;
		clock_gettime(CLOCK_REALTIME, &tps);

		for (iter = 0; iter < ITER; iter++)
			for (unsigned int i = 0; i < STEPS_N; i++) {
				/*First read into cache
				* make sure to access diff cache line.
				* Values are not important*/
				array[(i*CACHE_LN_SZ) & stride] *= 100;
			}

		clock_gettime(CLOCK_REALTIME, &tpe);
		retval = ((tpe.tv_sec-tps.tv_sec)*1000000000  + tpe.tv_nsec-tps.tv_nsec)/1000;

		if(index > 0){
			diff = retval - prev;
			if(max < diff) {
				max= diff;
				maxidx =j;
			}
		}
		
		//cout << "LLCCacheSizeTest "<< retval <<" stride "<<j<< " diffs: "<<diff<<endl;
		prev = retval;
		if(index == 0) j = MB;
		else j = j + (MB);
		index++;
	}

	free(array);
	cout<<"Effective LLC Cache Size "<<maxidx/MB<<" MB"<<endl;
}
```

### 3.Cache line:
现代CPU访问某个内存地址之后会把该内存地址周边的相邻地址的内容以cache line为单位导入缓存，cache line的一般容量是64字节，也就是16个整型。以下代码可以用来验证cache line的容量。基本思路以及原理同cache容量的测试，此处不赘述。

```c++
#include "stdafx.h"
#include <iostream>
#include <chrono>
#include <math.h>
using namespace std::chrono;

const int buff_size = 32 * 1024 * 1024;

void test_cache_hit(int * buffer, int size, int step)
{
	for (int i = 0; i < size;) {
		buffer[i] *= 3;
		i += step;
	}
}

int main()
{
	int * buffer = new int[buff_size]{};

	for (int i = 0; i < 16; ++i) {
		int step = (int)pow(2, i);

		auto start = std::chrono::system_clock::now();
		test_cache_hit(buffer, buff_size, step);
		auto end = std::chrono::system_clock::now();

		std::chrono::duration<double> elapsed_seconds = end - start;
		std::cout << "step: " << step << ", elapsed time: " << elapsed_seconds.count() * 1000 << "ms\n";
	}

	delete[] buffer;
}
```

下面是代码的运行结果：
step: 1, elapsed time: 66.3925ms

step: 2, elapsed time: 34.1113ms

step: 4, elapsed time: 26.5611ms

step: 8, elapsed time: 27.8086ms

step: 16, elapsed time: 27.8812ms

step: 32, elapsed time: 19.7227ms

step: 64, elapsed time: 11.3431ms

step: 128, elapsed time: 4.7402ms

step: 256, elapsed time: 2.4116ms

step: 512, elapsed time: 2.0155ms

step: 1024, elapsed time: 1.7971ms

step: 2048, elapsed time: 1.1497ms

step: 4096, elapsed time: 0.5761ms

step: 8192, elapsed time: 0.2719ms

step: 16384, elapsed time: 0.1615ms

step: 32768, elapsed time: 0.1099ms

Press any key to continue . . .

步长为2、4、8、16的时候耗时都差不多，这是因为cache line的容量是64字节，因此这几个步长对内存的访问量虽然大大不同，但是耗时都接近。值得注意的是，步长1和2的耗时差别相当大。我觉得cache line在这两个步长依然起作用的，只是两者的“计算消耗”耗时差别比较大，例如循环计数加法运算等等，所以1和2之间的差别可以理解为误差。

### 4. 多核cpu cache的false sharing问题
对于多核cpu，每个核都有一个cache。当多个线程访问同一个内存地址以及该地址的相邻地址的时候，如果这些地址可以被映射为同一个cache line的话，那么各个核的cache line需要同步，这可能导致程序的运行时间大大增加。这个问题称为 false sharing。

下面的代码来自这个链接：
https://vorbrodt.blog/2019/02/02/cache-lines/

```c++
#include "stdafx.h"
#include <iostream>
#include <thread>
#include <chrono>
#include <cstdlib>

using namespace std;
using namespace chrono;

const int CACHE_LINE_SIZE = 64;//sizeof(int);
const int SIZE = 2; //第一种方式
//const int SIZE = CACHE_LINE_SIZE / sizeof(int) + 1; //第二种方式
const int COUNT = 100'000'000;

int main(int argc, char** argv)
{
	srand((unsigned int)time(NULL));

	int* p = new int[SIZE];

	auto proc = [](int* data) {
	 for (int i = 0; i < COUNT; ++i)
	  *data = *data + rand();
	};

	auto start_time = high_resolution_clock::now();

	std::thread t1(proc, &p[0]);
	std::thread t2(proc, &p[SIZE - 1]);

	t1.join();
	t2.join();

	auto end_time = high_resolution_clock::now();
	cout << "Duration: " << duration_cast<microseconds>(end_time - start_time).count() / 1000.f << " ms" << endl;

	getchar();

	return 1;
}
```

在以上的代码中，我们可以试着用两种不同的size分配内存并测量运行时间，我们发现第一种方式和第二种方式有明显的性能差别。在我的电脑上，第一种方式耗时7030ms，第二种方式耗时3369ms。

第一种方式中，两个线程同时访问两个相邻的内存地址，并且这两个地址会被映射为同一个cache line，所以这两个线程会把同一段cache line的内存导入各自的cache。当一个线程修改一个内存地址的内容的时候，另一个线程不得不同步cache line，而从测试结果来看同步操作的成本相当高。

解决这个问题的方法就是避免两个线程访问的内存地址落入同一个cache line；第二种方式增加了两个相邻内存地址的步长，从而避免了false cache的问题。
