# 设置观察点只针对特定线程生效
## 例子
	#include <stdio.h>
	#include <pthread.h>
	
	int a = 0;
	
	void *thread1_func(void *p_arg)
	{
	        while (1)
	        {
	                a++;
	                sleep(10);
	        }
	}
	
	void *thread2_func(void *p_arg)
	{
	        while (1)
	        {
	                a++;
	                sleep(10);
	        }
	}
	
	int main(void)
	{
	        pthread_t t1, t2;
	
	        pthread_create(&t1, NULL, thread1_func, "Thread 1");
			pthread_create(&t2, NULL, thread1_func, "Thread 2");
	
	        sleep(1000);
	        return;
	}

## 技巧
gdb可以使用“`watch expr thread threadnum`”命令设置观察点只针对特定线程生效，也就是只有编号为`threadnum`的线程改变了变量的值，程序才会停下来，其它编号线程改变变量的值不会让程序停住。以上面程序为例:  

	(gdb) start
	Temporary breakpoint 1 at 0x4005d4: file a.c, line 28.
	Starting program: /data2/home/nanxiao/a
	[Thread debugging using libthread_db enabled]
	Using host libthread_db library "/lib64/libthread_db.so.1".
	
	Temporary breakpoint 1, main () at a.c:28
	28              pthread_create(&t1, NULL, thread1_func, "Thread 1");
	(gdb) n
	[New Thread 0x7ffff782c700 (LWP 7746)]
	29                      pthread_create(&t2, NULL, thread1_func, "Thread 2");
	(gdb) n
	[New Thread 0x7ffff6e2b700 (LWP 7765)]
	31              sleep(1000);
	(gdb) i threads
	  Id   Target Id         Frame
	  3    Thread 0x7ffff6e2b700 (LWP 7765) 0x00007ffff7915911 in clone () from /lib64/libc.so.6
	  2    Thread 0x7ffff782c700 (LWP 7746) 0x00007ffff78d9bcd in nanosleep () from /lib64/libc.so.6
	* 1    Thread 0x7ffff7fe9700 (LWP 7716) main () at a.c:31
	(gdb) wa a thread 2
	Hardware watchpoint 2: a
	(gdb) c
	Continuing.
	[Switching to Thread 0x7ffff782c700 (LWP 7746)]
	Hardware watchpoint 2: a
	
	Old value = 1
	New value = 3
	thread1_func (p_arg=0x400718) at a.c:11
	11                      sleep(10);
	(gdb) c
	Continuing.
	Hardware watchpoint 2: a
	
	Old value = 3
	New value = 5
	thread1_func (p_arg=0x400718) at a.c:11
	11                      sleep(10);
	(gdb) c
	Continuing.
	Hardware watchpoint 2: a
	
	Old value = 5
	New value = 7
	thread1_func (p_arg=0x400718) at a.c:11
	11                      sleep(10);


可以看到，使用“`wa a thread 2`”命令（`wa`是`watch`命令的缩写）以后，只有`thread1_func`改变`a`的值才会让程序停下来。  
参见[gdb手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Watchpoints.html).

## 贡献者

nanxiao