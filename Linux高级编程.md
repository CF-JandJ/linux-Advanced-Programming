[toc]


# 1.程序进程线程概念_进程ID号

## 相应概念

进程：

线程：

程序：

任务：

## 查看手册

查看手册:`man xxx`

查看获取进程号： `man getpid`

## 获取进程号(PID号)

### 用法
```
#include <sys/types.h>
#include <unistd.h>

pid_t getpid(void);
pid_t getppid(void);

//如何理解pid_t 其实就是typedef int pid_t
```

### 描述

`getpid()` : 返回当前调用进程的PID号

`getppid()`：返回当前调用进程的父进程的PID号

### 实例

#### 如何获取PID号
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main(void)
{
	pid_t pid;

	while(1)
	{
		printf("pid = %d\n", getpid());
		printf("ppid = %d\n", getppid());
		printf("Hello world\n");
		sleep(1);
	}

	return 0;
}

```


## 获得当前进程树

列出当前进程树：`pstree -p`

> 注意：systemd(init)：所有进程的父进程 pid =1

# 2.进程创建函数

## **fork 函数**

### 目的：

创建一个子进程

### 用法：
```
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
```

### 描述：
1. `fork()`函数创建一个新的进程通过拷贝调用进程
2. 新创建出来的进程称为子进程，调用`fork()`函数的进程称为父进程
3. 子进程拷贝了父进程的内容
4. 子进程和父进程相互运行在彼此分离的内存空间中
5. 返回值：如果创建成功，父进程返回的是子进程的PID号，子进程返回0；如果创建失败，父进程返回-1，没有子进程被创建

### 实例

#### 父子进程返回PID值

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main(){
        pid_t pid;
        pid = fork();
        printf("pid = %d\n",pid);
        printf("Hello world\n");

        return 0;
}
```

![image.png](https://note.youdao.com/yws/res/18888/WEBRESOURCE19d970556d39e5f3de6573124db676e3)

根据：返回值：如果创建成功，父进程返回的是子进程的PID号，子进程返回0；如果创建失败，父进程返回-1，没有子进程被创建

#### 同时调用2个`fork`函数，此时有多少个进程执行，其PID号相应是多少

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main(void)
{
	pid_t pid1, pid2;

	pid1 = fork();
	pid2 = fork();

	printf("pid1 = %d, pid2 = %d\n", pid1, pid2);

	return 0;
}

```

- 执行结果：

![image.png](https://note.youdao.com/yws/res/18894/WEBRESOURCEb170c96285f545f684148eabd7e5c7f9)


分析图示：

![image.png](https://note.youdao.com/yws/res/18896/WEBRESOURCE3ed39529da0522c4c4eec35211e6115b)

1. A进程：61580 , 61581
2. B进程：0 , 61582
3. C进程：61580 , 0
4. D进程：0 , 0

> 为何C进程是61580,0 ；C进程明明没有执行pid1的`fork()`函数？

因为子进程会拷贝父进程的内容，所以C进程拷贝了A进程的pid1，同理D进程

> 进程号怎么对应？

第一次调用`fork()`函数，此时`pid1`为子进程的`pid`号，所以`B`进程的`pid`号为`61580`，第二次调用`fork()`函数，此时`pid2`为父进程`A`的子进程`C`，`pid`号为`61581`。当用`B`进程调用`fork()`函数时，此时返回的`pid2`为`B`进程的子进程`D`，`pid`号为`61582`。


每次运行的结果，其`PID`号都不一样，因为由操作系统分配，后台有进程运行，结束后进程`ID`号释放掉，再调用会重新分配

## 父子进程执行不同语句

由于父子进程返回值不同，所以可以考虑用if语句来分别执行

### 实例

#### 父子进程分开执行，操作同一个变量，会不会相互影响

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

//int count = 0; (1

int main(void)
{
	pid_t pid;
//	int count = 0; (2

	pid = fork();

	if(pid > 0)  //parent process
	{
		while(1)
		{
			printf("Hello world, count = %d\n", count++);
			sleep(1);
		}
	}
	else if(pid == 0)  //child process
	{
		while(1)
		{
			printf("Good morning, count = %d\n", count++);
			sleep(2);
		}
	}
	else
		perror("fork");

	return 0;
}

```

> 父进程，子进程哪个先被调用？

Linux2.6以后，微观看默认父进程比子进程先调用，但是不能单看printf哪个先打印出来，因为printf是一个函数，父进程先调用，不一定意味着先打印出结果

- 执行结果：

按步骤2)执行：

![image.png](https://note.youdao.com/yws/res/18946/WEBRESOURCEa09cda268ea72199d39c6910166b7e9d)

此时父进程count数为：0-1-2-3-4-5-6-7...，子进程count数为：0-1-2-3-4-5...

故可以看出，父进程和子进程打印count值不受对方的影响。

> 为什么？

因为父进程和子进程是存储在不同的内存空间中，虽然变量名相同，但是值不同。所以不会相互影响

![image.png](https://note.youdao.com/yws/res/18962/WEBRESOURCE844141974966600c2419ea8f1e367f9e)

按步骤1)执行：结果一样

因为只不过是存储在局部区和全局区的区别，子进程和父进程还是属于不同的存储空间

#### 父进程和子进程之间，运行状态会不会互相影响

<span id = "多进程输出变量"></span>

[to mutithread](#多线程输出变量)

> 父进程结束，子进程如何？

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int count = 0;

int main(void)
{
	pid_t pid;
	int i;

	pid = fork();

	if(pid > 0)  //parent process
	{
		for(i = 0; i < 5; i++)//while(1)
		{
			printf("Hello world, count = %d\n", count++);
			sleep(1);
		}
	}
	else if(pid == 0)  //child process
	{
		while(1)
		{
			printf("Good morning, count = %d\n", count++);
			sleep(1);
		}
	}
	else
		perror("fork");

	return 0;
}
```

- 结果：

![image.png](https://note.youdao.com/yws/res/18987/WEBRESOURCE0364e3577ae174a9b44355d707260754)

父进程结束，子进程仍然继续在运行！

> 为什么？

父进程和子进程都是相互独立的内存空间，资源都不相互依赖，所以父进程结束，丝毫不影响子进程。

同理，子进程结束，不影响父进程


# 3.监控子进程wait函数

## `wait()`函数

wait , waitpid , waitid
等待进程改变其状态 
等待任意子进程终止

暂时只介绍wait

### 用法

```c
#include <sys/types.h>
#include <sys/wait.h>

pid_t wait(int *wstatus);

pid_t waitpid(pid_t pid,int * wstatus, int options)

pid_t waitid(idtype_t idtype,id_t id,siginfo_t * infop ,int options)

```

### 描述

1. `wstatus`是一个输出值，子进程状态改变信息，不需要则直接设置空指针即可
1. 返回值：终止进程的pid，当所有子进程都运行结束了，此时返回-1，表明调用进程并无未被等待的进程

### 实例

#### 创建3个子进程，不同时间结束

```c
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

//run model: ./a.out 10 5 15  (three child process, after 10, 5, 15seconds, they are over)
int main(int argc, char *argv[])
{
	pid_t child_pid;
	int numDead;
	int i;

	for(i = 1; i < argc; i++)
	{
		switch(fork())
		{
			case -1:
				perror("fork()");
				exit(0);
			case 0:
				printf("Child %d started with PID = %d, sleeping %s seconds\n", i, getpid(), argv[i]);
				sleep(atoi(argv[i]));
				exit(0);
			default:
				break;
		}
	}

	numDead = 0;

	while(1)
	{
		child_pid = wait(NULL);

		if(child_pid == -1)
		{
			printf("No more children, Byebye!\n");
			exit(0);
		}

		numDead++;
		printf("wait() returned child PID : %d(numDead = %d)\n", child_pid, numDead);
	}
}

```

- 结果：

![image.png](https://note.youdao.com/yws/res/19088/WEBRESOURCEe6daf465a1e15e979b6e394a364155b5)


# 4.创建线程函数

## `pthread_create()`函数

### 目的

创建一个新的线程

### 用法

```
#include <pthread.h>

int pthread_create(pthread_t *thread, const pthread_attr_t *attr,void *(*start_routine) (void*), void *arg);


编译链接时，带上  -pthread
```

### 描述

1. `thread`存放线程的ID号
2. `attr`是一个结构体指针，用来决定新线程的属性
1. 在调用的进程里，调用这个函数，会开始一个线程，新的线程开始执行`start_routine`；
2. 函数指针指向的函数，就是新创建的线程指向的内容
3. arg为传递线程函数的参数
4. 返回值：成功返回0，失败则返回一个错误值


### 实例

#### 创建线程

```C
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void *thread_function(void *arg);

int main(void)
{
	pthread_t pthread;
	int ret;
	int count = 5;

	ret = pthread_create(&pthread, NULL, thread_function, &count);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	pthread_join(pthread, NULL);（1
	printf("The thread is over, process is over too.\n");

	return 0; 
}

void *thread_function(void *arg)
{
	int i;
	printf("Thread begins running\n");

	for(i = 0; i < *(int *)arg; i++)
	{
		printf("Hello world\n");
		sleep(1);
	}
	return NULL;
}

```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/19251/WEBRESOURCE5f6fdebff1900bd5dc909dc12d675f36)

> 为何要加步骤1

线程用到的所有资源都来自于进程，如果没有加步骤1 ，则进程运行结束，操作系统回收所有进程的系统资源和内存空间，所以此时线程无法运行。

增加步骤1，进程能够检测到线程结束，等待线程结束，主进程再结束

## `pthread_join()`函数

### 目的

阻塞进程，等待线程结束

### 用法

```
#include <pthread.h>

int pthread_join(pthread_t thread, void ** retval);

Compile and link wiht  -pthread
```

### 描述

1. 等待创建的线程结束，如果线程已经结束，则将立刻返回；否则将一直阻塞等待线程结束
2. thread为要等待结束的线程ID
3. retval如果不是NULL，则保存目标线程退出的状态
4. 返回值：调用成功返回0；否则返回一个错误值

## `pthread_exit()`函数

### 目的

终止线程

### 用法

```c
#include <pthread.h>

void pthread_exit(void *retval);

//link with -pthread
```

# 5.多线程及线程间数据共享

### 实例

#### 创建多个线程,并且共享数据

<span id = "多线程输出变量"></span>

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void *thread1_function(void *arg);
void *thread2_function(void *arg);

int count = 0;

int main(void)
{
	pthread_t pthread1, pthread2;
	int ret;

	ret = pthread_create(&pthread1, NULL, thread1_function, NULL);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	ret = pthread_create(&pthread2, NULL, thread2_function, NULL);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	pthread_join(pthread1, NULL);
	pthread_join(pthread2, NULL);
	printf("The thread is over, process is over too.\n");

	return 0;
}

void *thread1_function(void *arg)
{
	printf("Thread1 begins running\n");

	while(1)
	{
		printf("Thread1 count = %d\n", count++);
		sleep(1);
	}
	return NULL;
}

void *thread2_function(void *arg)
{
	printf("Thread2 begins running\n");
	while(1)
	{
		printf("Thread2 count = %d\n", count++);
		sleep(1);
	}
	return NULL;
}

```

- 运行结果

![image.png](https://note.youdao.com/yws/res/19258/WEBRESOURCE33ee9e66799f2a0fdd17e2455940bb2a)

对比多进程输出变量：`count`的输出结果 [to](#多进程输出变量)

> 为什么会出现这种情况？

因为多线程中，count的值对所有线程都是可见的，每个线程的资源都是来自于进程，所以线程之间可以互相看到对方的数据。

# 任务间通信和同步方式

任务间通信和同步的方式：管道，信号，信号量，互斥锁，消息队列，共享内存，socket套接字

# 6.任务间通信方式

## 无名管道

### 函数 `pipe()`

#### 目的

创建管道，注意：只适用于有亲缘关系的进程，例父子进程，兄弟进程，为何？ [to](#不能使用无名管道)

<span id = "转到无名管道"></span>

#### 用法

```
#include <unistd.h>

int pipe(int pipefd[2]);
```

#### 描述

1. 半双工通信
2. 用于创建一个管道，数据间的通道可以用来进行任务间（interprocess）的通信。
3. `pipefd`输出类型参数，返回两个指向管道两端的文件描述符。(注意：进行通信，两个进程各一个管道)
4. `pipefd[0]`返回管道的读取端，`pipefd[1]`返回管道的写入端。
5. 往管道写入的数据被缓存在内核中，直到读取管道
6. 返回值：成功返回0，失败返回-1

![image.png](https://note.youdao.com/yws/res/19348/WEBRESOURCE54ca13d75eec501e8a01bb79ff03275c)

### 实例

#### 父进程写管道，子进程读管道

> 如何做？

1. 由于半双工，所以父进程打开写管道，关闭读管道，子进程打开读管道，关闭写管道。
2. 可以先调用`pipe()`函数建立管道，再调用`fork()`函数，此时子进程会复制父进程的管道，再进行步骤1的操作

```c
/*
	parent process: write pipe
	child  process: read  pipe
*/
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>

int main(void)
{
	int fd[2];
	int pid;

	if(pipe(fd) == -1)
		perror("pipe");

	pid = fork();
	if(pid > 0)  //parent process
	{
		close(fd[0]);
		sleep(5);
		write(fd[1], "ab", 2);

		while(1);
	}
	else if(pid == 0)
	{
		char ch[2];

		printf("Child process is waiting for data: \n");
		close(fd[1]);
		read(fd[0], ch, 2); // (1
		printf("Read from pipe: %s\n", ch);
	}

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/19354/WEBRESOURCEfe1b9461461a1888b4719b571eb9b24e)

> 如果一直没有数据，步骤1会如何？

步骤1会一直阻塞，直至读取完数据

#### 测试无名管道的大小

> 如何做？

父进程一直往管道写，子进程等待父进程结束

```c
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <sys/wait.h>

int main(void)
{
	pid_t pid;
	int fd[2];

	if(pipe(fd) == -1)
		perror("pipe");

	pid = fork();

	if(pid == 0)  //child process: write to the pipe
	{
		char ch = '*';
		int n = 0;

		close(fd[0]);

		while(1)
		{
			write(fd[1], &ch, 1);
			printf("count = %d\n", ++n);
		}
	}
	else if(pid > 0)  //parent process: wait until child process is over
	{
		waitpid(pid, NULL, 0);
	}
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/19364/WEBRESOURCEad1bf377bd1538aefa7bbfd8b29e124c)

说明管道最大容量65536字节。


#### 子进程写入键盘输出的字符串，父进程一直读取管道

```c
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <sys/wait.h>

int main(void)
{
	pid_t pid;
	int fd[2];

	if(pipe(fd) == -1)
		perror("pipe");

	pid = fork();

	if(pid == 0)
	{
		char tmp[100];

		close(fd[0]);

		while(1)
		{
			scanf("%s", tmp);
			write(fd[1], tmp, sizeof(tmp));
		}
	}
	else if(pid > 0)
	{
		char tmp[100];
		close(fd[1]);

		while(1)
		{
			printf("Parent process is waiting for the data from pipe:\n");
			read(fd[0], tmp, sizeof(tmp));
			printf("read from pipe: %s\n", tmp);
		}
	}

	return 0;
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/19374/WEBRESOURCE7245e73548e71f3dcb0f92e74a0ff63a)


#### 两条管道双向传输，父子进程相互传输

> 父进程将键盘的输入传递给子进程，子进程收到后，转为大写，再传回父进程

```c
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <sys/wait.h>
#include <string.h>
#include <ctype.h>

int main(void)
{
	pid_t pid;
	int fd[2];
	int fd2[2];

	if(pipe(fd) == -1)
		perror("pipe");
	if(pipe(fd2) == -1)
		perror("pipe");

	pid = fork();

	if(pid == 0)
	{
		char tmp[100];
		int i;

		close(fd[1]);//关闭写端
		close(fd2[0]);//关闭读端

		while(1)
		{
			memset(tmp, '\0', sizeof(tmp));
			read(fd[0], tmp, sizeof(tmp));

			for(i = 0; i < sizeof(tmp); i++)
				tmp[i] = toupper(tmp[i]);

			write(fd2[1], tmp, sizeof(tmp));
		}
	}
	else if(pid > 0)
	{
		char tmp[100];
		close(fd[0]);
		close(fd2[1]);

		while(1)
		{
			memset(tmp, '\0', sizeof(tmp));
			gets(tmp);
			write(fd[1], tmp, sizeof(tmp));

			memset(tmp, '\0', sizeof(tmp));
			read(fd2[0], tmp, sizeof(tmp));
			printf("After change: %s\n", tmp);
		}
	}

	return 0;
}
```

- 输出结果
 
![image.png](https://note.youdao.com/yws/res/19393/WEBRESOURCE781f5a2eca9f16a5be2581ce4c99b03c)


## 有名管道

<span id = "不能使用无名管道"></span>

[to 无名管道](#转到无名管道)

> 能否让两个没有亲缘关系的进程，使用无名管道进行通信？

什么意思？a.c和b.c，编译成可执行文件，是否能通过无名管道，使其进行通信

在a和b里面创建的`pipefd`数组，很明显不是同一个，不是全局变量

### mkfifo()函数

#### 目的

创建一个先入先出的特殊文件，即有名管道

#### 用法

```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname,mode_t mode);

```

#### 描述

1. 会创建一个先入先出的FIFO特殊文件，以pathname为文件名，
2. mode 用来设置FIFO的权限：读 2，写 4，可执行权限 1，三类用户
3. 只要创建了先入先出的FIFO特殊文件，任何进程都能打开进行读取或写入，和操作文件一样
4. 在进行输入输出之前，必须将文件的两端都同时打开
5. 当打开FIFO文件进行读时，会进行阻塞，直到有其他进程打开这个文件进行写
6. 返回值：成功返回0；失败返回-1


### 实例

#### 两个不相关的进程进行通信

> 怎么做？

创建两个.c文件，进行有名管道通信，一个.c文件进行读，一个.c文件进行写，写的内容以命令行参数形式

> read_named_pipe.c

```c
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

int main(void)
{
	int ret;
	int fd;
	char buf[100];

	ret = mkfifo("my_fifo", 666);
	if(ret != 0)
		perror("mkfifo");

	printf("Prepare reading from named pipe:\n");

	fd = open("my_fifo", O_RDWR);
	if(fd == -1)
		perror("open");

	while(1)
	{
		memset(buf, '\0', sizeof(buf));
		read(fd, buf, sizeof(buf));
		printf("Read from named pipe: %s\n", buf);
		sleep(1);
	}

	return 0;
}
```

> write_named_pipe.c

```c
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	int fd;
	char buf[100];

	fd = open("my_fifo", O_WRONLY);
	if(fd == -1)
		perror("open");

	if(argc == 1)
	{
		printf("Please send something to the named pipe:\n");
		exit(EXIT_FAILURE);
	}

	strcpy(buf, argv[1]);
	write(fd, buf, sizeof(buf));
	printf("Write to the pipe: %s\n", buf);

	return 0;
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/19500/WEBRESOURCE40187c25976dc995b3e20b122f4a41cf)

![image.png](https://note.youdao.com/yws/res/19498/WEBRESOURCEcca39350f5ab358d75aa01acc485a5d7)

## 共享内存

<span id = "共享内存方法"></span>

将共享内存映射到各个进程中内存空间中，各个进程操作本进程的映射内存空间，相当于操作共享内存。

![image.png](https://note.youdao.com/yws/res/19523/WEBRESOURCEe46ad95308549134f5e656a33a561edc)

需要进行的操作：

1. 创建共享内存
2. 在不同的进程中，将共享内存映射到当前的映射内存空间中
3. 解除映射绑定的关系
4. 删除映射内存空间


### `shmget()`函数

<span id = "shmget函数用法"></span>

#### 目的

开辟一个`system V`版本的共享内存单元

#### 用法

```c
#include <sys/ipc.h>
#include <sys/shm.h>

int shmget(key_t key, size_t size, int shmflg);
```
<span id = "shmflg标志符"></span>
<span id = "shm的key值"></span>

#### 描述

1. 返回值：返回一个`System v`版本的共享内存单元的标识符
2. `key`值与共享内存单元的标识符是关联在一起的
3. `key`值是在内核层标识共享内存单元，应用层用的是共享内存单元的标识符(即返回值)
4. `size`开辟出来的共享内存单元的大小，一般以字节为单位
5. 当把`key`值设置为`IPC_PRIVATE`时，由内核层分配`key`值
6. shmflg的值包括：`IPC_CREAT`、`IPC_EXCL`，`SHM_HUGETLB`

`IPC_CREAT`:如果不存在为`key`值的共享内存单元，则创建一个新的共享内存单元；如果存在，则查看是否有权限访问。

`IPC_EXCL`:和`IPC_CREAT`一起使用，用来确保创建共享内存，如果存在，则创建失败。

### `shmat()`函数

#### 目的

将`shmid`标识的共享内存映射到调用进程的内存地址空间中

#### 用法

```c
#include <sys/types.h>
#include <sys/shm.h>

void *shmat(int shmid,const void *shmaddr,int shmflg);
```

#### 描述

1. `shmid`：为要映射的共享内存
2. `shmaddr`：将要映射的共享内存放到该地址中。如果为`NULL`,系统会选用合适的地址来映射共享内存
3. `shmflg`：可以选择以下值

`SHM_EXEC`:映射的内存空间具有可执行权限
`SHM_RDONLY`:映射的内存空间只读权限
如果第三个参数不去配置(例可写为0)，那么具备读写权限

4. 返回值：成功则返回当前进程的映射内存空间的地址

### `shmdt()`函数

#### 目的

解除当前进程映射的地址单元和共享内存单元的绑定

#### 用法

```
#include <sys/types.h>
#include <sys/shm.h>

int shmdt(const void *shmaddr);
```

#### 描述

1. 需要解除的当前进程映射的地址单元的地址

### `shmctl()`函数

<span id = "shmctl函数"></span>

#### 目的

控制`system v`版本的共享内存

#### 用法

```c
#include <sys/ipc.h>
#include <sys/shm.h>

int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

#### 描述

1. `cmd`命令的值中`IPC_RMID`删除共享内存
2. `buf`默认为`NULL`

### 实例

#### 利用共享内存单元实现有亲缘关系的不同进程的共享

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

char msg[] = "Hello world";

int main(void)
{
	int shmid;
	pid_t pid;

	shmid = shmget(IPC_PRIVATE, 1024, IPC_CREAT);

	pid = fork();

	if(pid > 0)
	{
		char *p_addr;
		p_addr = shmat(shmid, NULL, 0);

		memset(p_addr, '\0', sizeof(msg));
		memcpy(p_addr, msg, sizeof(msg));

		shmdt(p_addr);

		waitpid(pid, NULL, 0);
	}
	else if(pid == 0)
	{
		char *c_addr;
		c_addr = shmat(shmid, NULL, 0);

		printf("Child process waits a short time: \n");
		sleep(3);
		printf("Child Process reads from shared memory: %s\n", c_addr);
		shmdt(c_addr);
	}
	else
		perror("fork");

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/19739/WEBRESOURCEa364aac42b86d5183310f5f64d28391d)


#### 非亲缘关系进程通过共享内存通信

> 如何才能做到？

利用共享内存内核分配的`key`值，不同进程的`key`值设置为同一个，即可。

> write_shm.c

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

char msg[] = "Hello world";

#define MY_KEY	9527

int main(void)
{
	int shmid;

	shmid = shmget(MY_KEY, 1024, IPC_CREAT);

	char *p_addr;
	p_addr = shmat(shmid, NULL, 0);

	memset(p_addr, '\0', sizeof(msg));
	memcpy(p_addr, msg, sizeof(msg));

	shmdt(p_addr);

//	shmctl(shmid, IPC_RMID, NULL);

	return 0;
}
```

> read_shm.c

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

#define MY_KEY	9527

int main(void)
{
	int shmid;

	shmid = shmget(MY_KEY, 1024, IPC_CREAT);

	char *c_addr;
	c_addr = shmat(shmid, NULL, 0);

	printf("Read from shared memory: %s\n", c_addr);
	shmdt(c_addr);

	return 0;
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/19788/WEBRESOURCE76bdc9fbe8a72b798a31931028ab1892)

![image.png](https://note.youdao.com/yws/res/19786/WEBRESOURCEee342cd7c056c66a6baf16bb04963d27)

### 内存映射

#### mmap()函数

##### 目的

系统调用在调用进程的虚拟地址空间中创建一个新映射

##### 用法

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
```

##### 描述

1. `addr`：指定了映射被放置的虚拟地址，如果指定`NULL`,内核会映射选择一个合适的地址
2. `length`：指定了映射的字节数
3. `prot`：位掩码，指定了施加于映射之上的保护信息，其取值要么为`PROT_NONE`，要么为以下三个标记的组合

|值|描述|
|---|---|
|PROT_NONE|区域无法访问|
|PROT_READ|区域内容可读取|
|PROT_WRITE|区域内容可修改|
|PROT_EXEC|区域内容可执行|

4. `flags`：控制映射操作各个方面的选项的位掩码，值为如下：

`MAP_PRIVATE`：创建一个私有映射。区域中内容上所发生的变更对使用同一映射的其他进程是不可见的

`MAP_SHARED`：创建一个共享映射。区域中内容上所发生的变更对使用`MAP_SHARED`特性映射同一区域的进程是可见的。

除上述两种，可包含其他值(取`OR`)

`MAP_ANONYMOUS`：创建一个匿名映射（没有对应文件的一种映射）

5. `fd`：用于文件映射的。标识被映射文件的文件描述符。匿名映射设置为`-1`。
6. `offset`：用于文件映射的。指定了映射在文件中的起点，它必须是系统分页大小的倍数。匿名映射设置为`0`。
7. 返回值：成功返回新映射的起始地址。发送错误时，返回`MAP_FAILED`。

<span id = "内存映射方法"></span>

## 消息队列

### `msgget()`函数

#### 目的

得到一个`System V`版本的消息队列标识符

#### 用法

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgget(key_t key, int msgflg);
```

#### 描述

1. 返回值：函数会返回`System V`版本的消息队列的标识符，和`key`值相关联
2. `key`值：同上，想创建新的消息队列，`key`值可以设置为`IPC_PRIVATE`，由内核自动分配，具体[to](#shm的key值)
1. `msgflg`：值可取`IPC_CREAT`、`IPC_EXCL`，具体[to](#shmflg标志符)

### `msgsnd` `msgrcv`

#### 目的

消息队列的发送函数,接受函数。即`System V`版本的消息队列的操作

#### 用法

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);

ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```

#### 描述

> `msgsnd()`

1. 发送消息to消息队列，需要获得一定的权限
2. `msgid`：消息队列的描述符
3. `msgp`：指向用户自定义结构体的指针，按如下形式

```c
struct msgbuf{
    long mtype; //消息类型，必须大于0
    char mtext[1]; //字符数组，发送消息的内容
    ------ 可自己添加，该结构体除了消息类型，其他都是发送消息内容
}
```

4. `msgsz`：发送消息内容的大小
5. `msgflg`：消息队列的标志，没有特别要求，可取 0 

> `msgrcv()`

1. 接受消息from消息队列，需要获得一定的权限
2. 相应参数同上`msgsnd()`
3. `msgtyp`：如果为 0 ，则接收所有`msgtyp`值的消息队列的消息，一般和发送消息的`mtype`相同

### `msgctl()`函数

#### 目的

`System V`消息控制操作，删除创建的消息队列

#### 用法

```
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgctl(int msqid, int cmd ,struct msqid_ds *buf);
```

#### 描述

1. 和`shmctl()`函数类似， [to](#shmctl函数)

### 实例

#### 利用消息队列实现有亲缘关系的不同进程的通信


```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#include <sys/msg.h>

#define MY_TYPE  9527

int main(void)
{
	int msgid;
	pid_t pid;

	struct msgbuf
	{
		long mtype;
		char mtext[100];
		int number;
	};

	struct msgbuf buff;

	msgid = msgget(IPC_PRIVATE, IPC_CREAT);

	pid = fork();

	if(pid > 0)
	{
		sleep(5);

		buff.mtype = MY_TYPE;
		printf("Please enter a string you want to send:\n");
		gets(buff.mtext);
		printf("Please enter a nubmer you want to send:\n");
		scanf("%d", &buff.number);

		msgsnd(msgid, &buff, sizeof(buff) - sizeof(buff.mtype), 0);

		waitpid(pid, NULL, 0);
	}
	else if(pid == 0)
	{
		printf("Child process is waiting for msg:\n");
		msgrcv(msgid, &buff, sizeof(buff) - sizeof(buff.mtype), MY_TYPE, 0);
		printf("Child process read from msg: %s, %d\n", buff.mtext, buff.number);
		msgctl(msgid, IPC_RMID, NULL);
	}
	else
		perror("fork");

	return 0;
}
```


- 运行结果：

![image.png](https://note.youdao.com/yws/res/20039/WEBRESOURCE98bd6c66f05278082c5f0c5cdc35751e)

> 为何`buff`即被父进程使用，又被子进程使用

因为子进程会拷贝父进程的`buff`，但此时父子进程中不是同一个

> 如果父子进程中的`typeid`不一致会如何？

如果父子进程中的`typeid`不一致，那么将接收不到消息

#### 非亲缘关系进程通过消息队列通信

> 如何进行？

同共享内存一样，如果非亲缘关系通信，发送端和接收端的`key`值都设置为`IPC_PRIVATE`，由于非亲缘关系，都是进行创建，由内核分配不同的`key`。所以可以手动设置一个`key`值，在非亲缘关系进程中使用相同`key`值

> send_msg.c

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#include <sys/msg.h>

#define MY_TYPE  9527
#define MY_KEY	 1314

int main(void)
{
	int msgid;

	struct msgbuf
	{
		long mtype;
		char mtext[100];
		int number;
	};

	struct msgbuf buff;

	msgid = msgget(MY_KEY, IPC_CREAT); //ftok

	buff.mtype = MY_TYPE;
	printf("Please enter a string you want to send:\n");
	gets(buff.mtext);
	printf("Please enter a nubmer you want to send:\n");
	scanf("%d", &buff.number);

	msgsnd(msgid, &buff, sizeof(buff) - sizeof(buff.mtype), 0);

	return 0;
}
```

> recv_msg.c

```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#include <sys/msg.h>

#define MY_TYPE  9527
#define MY_KEY	 1314

int main(void)
{
	int msgid;

	struct msgbuf
	{
		long mtype;
		char mtext[100];
		int number;
	};

	struct msgbuf buff;

	msgid = msgget(MY_KEY, IPC_CREAT);

	while(1)
	{
		printf("Process is waiting for msg:\n");
		msgrcv(msgid, &buff, sizeof(buff) - sizeof(buff.mtype), MY_TYPE, 0);
		printf("Process read from msg: %s, %d\n", buff.mtext, buff.number);
	}
	msgctl(msgid, IPC_RMID, NULL);

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/20073/WEBRESOURCE0bdb9fd8e9f167f723d242ecf452e269)

![image.png](https://note.youdao.com/yws/res/20071/WEBRESOURCE043c0d5b050e860108a31c139dae2752)

> 如果此时`recv_msg.c`中将`msgtyp`设置为0，是否可以通信？

可以，设置为0，表示从消息队列中接收所有消息。
如果设置为非0值且不等于发送端的`msgtyp`，则无法接收

# 7.任务间同步方式

## 无名信号量

信号量：内核维护的正整数值（`≥0`）

> 概念

无名信号量：这些信号量没有名字，相反，它位于内存中一个预先商定的位置处。无名信号量可以在进程之间或一组线程之间共享。当在进程之间共享时，信号量必须位于一个共享内存区域中。当在线程之间共享时，信号量可以位于被这些线程共享的一块内存区域中。

### `sem_init()`函数

<span id = "sem_init()用法"</span>

#### 目的

`POSIX`信号量，对一个信号量进行初始化并通知系统该信号量会在进程间共享还是在单个进程中的线程间共享

#### 用法

```c
#include <semaphore.h>

int sem_init(sem_t *sem, int pshared, unsigned int value);

//LINK with -pthread
```

#### 描述

1. `sem`：表示指向信号量的指针
2. `pshared`：决定信号量是在进程间的线程共享（`value = 0`,且信号量要放在被线程共享的内存区域中），还是进程间共享（`value != 0`，且信号量要放在共享的内存区域中,[to共享内存](#共享内存方法)）
3. `value`：信号量初始值
1. 返回值：成功返回0，失败返回-1

### `sem_wait()`函数

#### 目的

锁定信号量,即等待信号量

#### 用法

```c
#include <semaphore.h>

int sem_wait(sem_t *sem);
```

#### 描述

1. 对`sem`的信号量进行`-1`操作。如果此时信号量的值`＞0`，则进行一次递减操作(即`-1`操作)；如果信号量的值`= 0 `，则调用进程将会被阻塞，直到当前信号量的值`>0`。
2. `sem`：表示指向信号量的指针
3. 返回值：成功返回0，失败返回-1


### `sem_post()`函数

#### 目的

解锁一个信号量

#### 用法

```c
#include <semaphore.h>

int sem_post(sem_t *sem);

//LINK with -pthread
```

#### 描述

1. 对`sem`的信号量进行`+1`操作。进行一次递增操作（即`+1`操作），如果信号量的值立马`>0`，其他被阻塞的调用进程/线程将被唤醒

### 实例

#### 具有亲缘关系的进程利用信号量同步

> 代码逻辑

父进程检查信号量，如果大于0，则打印一个提示信息，子进程每隔一段时间，发布一个信号量。

```c
#include <stdio.h>
#include <semaphore.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/mman.h>

int main(void)
{
	//sem_t *sem_id = NULL; //(1
	sem_t sem_id;
	pid_t pid;


	sem_init(&sem_id, 1, 1);

	pid = fork();

	if(pid > 0)
	{
		while(1)
		{
			sem_wait(&sem_id);
			printf("This is parent process.\n");
			sleep(1);
		}
	}
	else if(pid == 0)
	{
		while(1)
		{
			printf("This is child process.\n");
			sleep(5);
			sem_post(&sem_id);
		}
	}

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/20329/WEBRESOURCEa6a557da4348d5557b24ed28ac35bf22)

> 为什么用步骤1不行？

因为`sem_id`指向空，没有分配空间，无法存储

> 为何不执行父进程？

父进程和子进程都有独立的地址空间，创建的`sem_id`在父子进程中不是同一个，不在相同的地址空间中。

> 如何解决这个问题呢？

1. 将信号量放在共享内存中即可，利用`shmget()`函数[to](#shmget函数用法)
2. 创建共享映射，利用`mmap()`函数[to](#内存映射方法)



```c
#include <stdio.h>
#include <semaphore.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/mman.h>

int main(void)
{
	sem_t *sem_id = NULL;
	pid_t pid;

	sem_id = mmap(NULL, sizeof(sem_t), PROT_READ|PROT_WRITE, MAP_SHARED|MAP_ANONYMOUS, -1, 0);

	sem_init(sem_id, 1, 0);

	pid = fork();

	if(pid > 0)
	{
		while(1)
		{
			sem_wait(sem_id);
			printf("This is parent process.\n");
			sleep(1);
		}
	}
	else if(pid == 0)
	{
		while(1)
		{
			printf("This is child process.\n");
			sleep(5);
			sem_post(sem_id);
		}
	}

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/20546/WEBRESOURCEc0c5533f21f9ab624936bd70ef828514)

## 命名信号量

### `sem_open`函数

#### 目的

初始化并且打开一个命名信号量

#### 用法

```c
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>

sem_t *sem_open(const char *name, int oflag);
sem_t *sem_open(const char *name, int oflag, mode_t mode, unsigned int value);

//LINK with -pthread
```

#### 描述

1. 创建一个`POSIX`信号量或打开一个已经存在的信号量。
2. `name`：被用来识别信号量
3. `oflag`：被用来控制调用操作。有以下值：

`O_CREAT`：如果信号量不存在，则会创建信号量，如果使用该`value`，则需要附加两个参数。

`O_CREAT | O_EXCL`：如果该信号量存在，则返回一个错误

4. `mode`：用来配置新信号量的许可权限，可读(`2`)可写(`4`)可执行(`1`)，三类用户
5. `value`：信号量的初始值
6. 如果指定`O_CREAT`，且命名的信号量已经存在，则忽略第3和第4个参数。
7. 返回值：创建成功，返回新创建的信号量的地址，失败返回`SEM_FAILED`


### `sem_close()`函数

#### 目的

关闭一个信号量

#### 用法

```c
#include <semaphore.h>

int sem_close(sem_t *sem);

//LINK with -pthread
```

#### 描述

1. 用来关闭`sem`指向的信号量，当前进程中所有和信号量相关的资源都会被释放

### `sem_unlink()`函数

#### 目的

删除一个信号量

#### 用法

```c
#include <semaphore.h>

int sem_unlink(const char *name);

//LINK with -pthread
```

#### 描述

1. 删除指定为`name`的信号量

### 实例

#### 非亲缘关系的进程利用信号量同步

> `named_sem1.c`

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <semaphore.h>
#include <fcntl.h>
#include <sys/stat.h>

int main(void)
{
	sem_t *sem;
	int i = 0;

	sem = sem_open("NAMED_SEM", O_CREAT, 666, 0);

	while(1)
	{
		sem_wait(sem);
		printf("Process 1: i = %d\n", i++);
	}

	sem_close(sem);

	return 0;
}
```


> `named_sem2.c`

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <semaphore.h>
#include <fcntl.h>
#include <sys/stat.h>

int main(void)
{
	sem_t *sem;
	int i = 0;

	sem = sem_open("NAMED_SEM", O_CREAT, 666, 0);

	while(1)
	{
		printf("Process 2: i = %d\n", i++);
		sleep(1);
		sem_post(sem);
	}

	sem_close(sem);

	return 0;
}
```

- 运行结果：

先运行`named_sem2.c`

![image.png](https://note.youdao.com/yws/res/20710/WEBRESOURCE0eb2e6b90710d07d22689d8790906b54)

此时再运行`named_sem1.c`

![image.png](https://note.youdao.com/yws/res/20712/WEBRESOURCE1634afa3559a7f803af9997a22efa715)

> 为何？

因为`named_sem2.c`不断增加信号量，然后`named_sem1.c`减少信号量直至`0`。为何值不一样？因为`named_sem2.c`执行停止时，还未执行增加信号的函数即`sem_post()`

![image.png](https://note.youdao.com/yws/res/20717/WEBRESOURCEe5970449c807b4cc2b7fac89bbd3c865)

- 运行结果：

先运行`named_sem1.c`，再运行`named_sem2.c`，再运行`named_sem1.c`

![image.png](https://note.youdao.com/yws/res/20731/WEBRESOURCE206d1bbbd93516cacd255367e4965299)

![image.png](https://note.youdao.com/yws/res/20723/WEBRESOURCE27af04f6d0c45e7cf06b3ffdb3cef329)

![image.png](https://note.youdao.com/yws/res/20733/WEBRESOURCE4619aa08ecc698f7d979e09dbfdd30f7)

> 为何不一样？

因为此时关闭了进程，调用了`sem_close()`函数，释放了资源，所以停止。同理，当再次调用，被激活。


## 线程间信号量同步

创建信号量，且用于线程间通信 [to](#sem_init()用法)

### 实例

#### 线程间利用信号量同步

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <semaphore.h>

void *thread1_function(void *arg);
void *thread2_function(void *arg);

int count = 0;
sem_t sem;

int main(void)
{
	pthread_t pthread1, pthread2;
	int ret;

	sem_init(&sem, 0, 0);

	ret = pthread_create(&pthread1, NULL, thread1_function, NULL);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	ret = pthread_create(&pthread2, NULL, thread2_function, NULL);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	pthread_join(pthread1, NULL);
	pthread_join(pthread2, NULL);
	printf("The thread is over, process is over too.\n");

	return 0;
}

void *thread1_function(void *arg)
{
	while(1)
	{
		sem_wait(&sem);
		printf("Thread1 count = %d\n", count++);
	}
	return NULL;
}

void *thread2_function(void *arg)
{
	while(1)
	{
		printf("Thread2 is running!\n");
		sleep(5);
		sem_post(&sem);
	}
	return NULL;
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/20906/WEBRESOURCEe496cc96fc33418e0f3cc111d6bdbbe7)

> 为什么此时不用`mmap`？

因为线程共用进程的资源,所以不用映射内存单元


## 互斥锁

常常用在线程之间

### `pthread_mutex_init()函数`

#### 目的

操作互斥锁，用于互斥锁的初始化

#### 用法

```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr)
```

#### 描述

1. 一个互斥锁只能被一个线程拥有，不可能被两个或多个线程拥有。如果一个线程尝试对一个已经加锁的线程进行加锁，线程会被挂起直到拥有互斥锁的线程被解锁后，才能进行加锁。
2. 初始化`mutex`指向的互斥锁。
3. `nutexattr`：互斥锁的选项和配置。如果设置为`NULL`,将采用默认的配置。

### `pthread_mutex_lock()`函数和`pthread_mutex_unlock()`函数

#### 目的

对互斥锁进行上锁，对互斥锁进行解锁

#### 用法

```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);


pthread_mutex_lock(&mut);
/* operate on x */
pthread_mutex_unlock(&mut);
```

#### 描述

1. 互斥锁有两种可能的状态：加锁（被一个线程拥有）or解锁（不属于任何线程）


### 实例

#### 线程间利用互斥锁实现同步

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <semaphore.h>

void *thread1_function(void *arg);
void *thread2_function(void *arg);

pthread_mutex_t mutex;

int main(void)
{
	pthread_t pthread1, pthread2;
	int ret;

	pthread_mutex_init(&mutex, NULL);

	ret = pthread_create(&pthread1, NULL, thread1_function, NULL);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	ret = pthread_create(&pthread2, NULL, thread2_function, NULL);
	if(ret != 0)
	{
		perror("pthread_create");
		exit(1);
	}

	pthread_join(pthread1, NULL);
	pthread_join(pthread2, NULL);
	printf("The thread is over, process is over too.\n");

	return 0;
}

void *thread1_function(void *arg)
{
	int i;

	while(1)
	{
		pthread_mutex_lock(&mutex);
		for(i = 0; i < 2; i++)
		{
			printf("Hello world\n");
			sleep(1);
		}
		pthread_mutex_unlock(&mutex);
		sleep(1);
	}
	return NULL;
}

void *thread2_function(void *arg)
{
	int i;
	sleep(1);

	while(1)
	{
		pthread_mutex_lock(&mutex);
		for(i = 0; i < 2; i++)
		{
			printf("Good moring\n");
			sleep(1);
		}
		pthread_mutex_unlock(&mutex);
		sleep(1);
	}
	return NULL;
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/20910/WEBRESOURCE36a3f3f483e852e9e312297013b74e3f)


## 信号

往往用信号来传递控制命令，很少用来传递数据 

查看信号命令  `kill -l`

### `kill()`函数

#### 目的

向某个进程发送信号

#### 用法

```c
#include <sys/types.h>
#include <signal.h>

int kill(pid_t pid, int sig);
```

#### 描述

1. `pid`:如果`pid`为正值，向`pid`进程发送信号；如果`pid`为0值，会向进程组里的每一个进程发送信号；如果`pid`为-1值，向所有的进程发送信号
2. `sig`：信号
3. 返回值：成功为0，失败为-1

![image.png](https://note.youdao.com/yws/res/20946/WEBRESOURCE60405c3181cf3b7cd6ee68cff10443b5)


### 实例

#### 利用信号量传递进程间的控制信息

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <sys/types.h>

int main(void)
{
	pid_t pid;

	pid = fork();

	if(pid > 0)
	{
		sleep(5);
		kill(pid, SIGKILL);
	}
	else if(pid == 0)
	{
		int i = 0;
		printf("Child process id = %d\n", getpid());
		while(1)
		{
			printf("Count to %d\n", ++i);
			sleep(1);
		}
	}
	else
		perror("fork");

	return 0;
}
```

- 运行结果


![image.png](https://note.youdao.com/yws/res/20956/WEBRESOURCEbb56a273581c8128fd9be41bd2f76261)

### `signal`函数

#### 目的

信号处理函数

#### 用法

```c
#include <signal.h>

typedef void (*sighandler_t)(int);
sinhandler_t signal(int signum, sighandler_t handler);

```

#### 描述

1. 发送不同的信号，会调用不同的函数去处理
2. `signum`：信号(调用`kill -l`查看)
3. `handler`：信号处理函数


### 实例

#### 根据不同的信号量处理不同的函数

> 1signal.c

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <signal.h>

void function(int signo);

int main(void)
{
	int i = 0;

	printf("pid = %d\n", getpid());

	signal(SIGINT, function);
	signal(SIGKILL, function);

	while(1)
	{
		printf("Count to %d\n", ++i);
		sleep(1);
	}

	return 0;
}

void function(int signo)
{
	if(signo == SIGINT)
	{
		printf("You have just triggered a ctrl+c operation.\n");
		exit(1);
	}

	else if(signo == SIGQUIT)
		printf("Trig a SIGQUIT signal.\n");
}
```

- 运行结果

![image.png](https://note.youdao.com/yws/res/21022/WEBRESOURCE060319e85d482729a160285dc042fdb5)


> kill.c

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <signal.h>

int main(int argc, char *argv[])
{
	kill(atoi(argv[1]), SIGQUIT);

	return 0;
}
```

> 先编译`1signal.c`再编译`kill.c`传递信号

- 运行结果

![image.png](https://note.youdao.com/yws/res/21026/WEBRESOURCEd10519bdcc782b141da74d036ba9d1dc)

![image.png](https://note.youdao.com/yws/res/21028/WEBRESOURCE2789f01ef1875f7c84017827c7b86448)

> 为什么`"Trig a SIGQUIT signal.\n"`没有打印出来？

因为`SIGKILL`是不可修改，不可屏蔽的信号


# 网络编程

> 服务器端与客户端数据交互过程

![image.png](https://note.youdao.com/yws/res/21057/WEBRESOURCEca0592c71cb6e495284707d6fe6e4dff)


## TCP编程

### `socket()`函数

#### 目的

建立一个终端用于通信

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

#### 描述

详细见tcp/ip编程

### `bind()`函数

#### 目的

将一个名字绑定要套接字上

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

#### 描述

1. 创建一个套接字，会存在一个命名空间，但是没有和地址绑定到一起。
2. 用于将`addr`指向的地址和`sockfd`套接字绑定到一起


### `listen()`函数

#### 目的

监听套接字的连接

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

int listen (int sockfd, int backlog);
```

#### 描述

1. `backlog`：定义用于挂起套接字等待连接队列的最大长度


### `accept()`函数

#### 目的

接收一个套接字的连接

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

#### 描述

1. 此时`addr`是想访问服务器的客户端的地址，输出参数。（如果为服务器端使用）
2. 返回值：新的套接字描述符


### `close()`函数

关闭`sockfd`套接字连接

### `connect()`函数

#### 目的

初始化一个套接字连接

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

#### 描述

1. 连接一个套接字描述符到一个地址上


> server.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>

#define PORT_ID	8800
#define SIZE	100

int main(void)
{
	int sockfd, client_sockfd;
	struct sockaddr_in my_addr, client_addr;
	int addr_len;
	char welcome[SIZE] = "Welcome to connect to the sever!";

	//1.socket()
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.bind()
	my_addr.sin_family = AF_INET;
	my_addr.sin_port = htons(PORT_ID);
	my_addr.sin_addr.s_addr = INADDR_ANY;
	bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr));

	//3.listen()
	listen(sockfd, 10);

	addr_len = sizeof(struct sockaddr);

	while(1)
	{
		printf("Server is waiting for client to connect:\n");
		//4.accept()
		client_sockfd = accept(sockfd, (struct sockaddr *)&client_addr, &addr_len);
		printf("Client IP address = %s\n", inet_ntoa(client_addr.sin_addr));
		//5.send()
		send(client_sockfd, welcome, SIZE, 0);
		printf("Disconnect the client request.\n");
		//6.close()
		close(client_sockfd);
	}

	close(sockfd);

	return 0;
}
```

> client.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>

#define PORT_ID	8800
#define SIZE	100

//./client IP  : ./client 192.168.0.10
int main(int argc, char *argv[])
{
	int sockfd;
	struct sockaddr_in server_addr;
	char buf[SIZE];

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket()
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.connect()
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT_ID);
	server_addr.sin_addr.s_addr = inet_addr(argv[1]);
	connect(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));

	recv(sockfd, buf, SIZE, 0);
	printf("Client receive from server: %s\n", buf);

	close(sockfd);

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/21059/WEBRESOURCE56b657ff70588f4271320e974effaca6)

![image.png](https://note.youdao.com/yws/res/21060/WEBRESOURCE80faa42f942170a3525b1af4ce3b41df)


## UDP编程

> 服务器端与客户端数据交互过程


![image.png](https://note.youdao.com/yws/res/21061/WEBRESOURCE18957f17769b5a02ac7e5a592538d574)

### `recvfrom()`函数

#### 目的

从套接字中接受一个信息

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t recvform (int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
```

#### 描述

1. 此时`addr`是想访问服务器的客户端的地址，输出参数。（如果为服务器端使用）


### `sendto()`函数

#### 目的

从套接字中发送一个信息

#### 用法

```c
#include <sys/types.h>
#include <sys/socket.h>

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
```

#### 描述

1. `dest_addr`：要发送的服务器的地址（如果为客户端使用）

> server.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>

#define PORT_ID	8800
#define SIZE	100

int main(void)
{
	int sockfd;
	struct sockaddr_in my_addr, client_addr;
	int addr_len;
	char buf[SIZE];

	//1.socket()
	sockfd = socket(AF_INET, SOCK_DGRAM, 0);

	//2.bind()
	my_addr.sin_family = AF_INET;
	my_addr.sin_port = htons(PORT_ID);
	my_addr.sin_addr.s_addr = INADDR_ANY;
	bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr));

	addr_len = sizeof(struct sockaddr);

	while(1)
	{
		printf("Server is waiting for client to connect:\n");
		//4.recv()
		recvfrom(sockfd, buf, SIZE, 0, (struct sockaddr *)&client_addr, &addr_len);
		printf("Server receive from client: %s\n", buf);
	}

	close(sockfd);

	return 0;
}
```

> client.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>

#define PORT_ID	8800
#define SIZE	100

//./client IP  : ./client 192.168.0.10
int main(int argc, char *argv[])
{
	int sockfd;
	struct sockaddr_in server_addr;
	char buf[SIZE];
	int i;

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket()
	sockfd = socket(AF_INET, SOCK_DGRAM, 0);

	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT_ID);
	server_addr.sin_addr.s_addr = inet_addr(argv[1]);

	for(i = 0; i < 10; i++)
	{
		sprintf(buf, "%d\n", i);
		sendto(sockfd, buf, SIZE, 0, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));
		printf("Client sends to server %s: %s\n", argv[1], buf);
		sleep(1);
	}

	close(sockfd);

	return 0;
}
```


- 运行结果：

![image.png](https://note.youdao.com/yws/res/21062/WEBRESOURCEe6c5352466351bbadd1eecbc695f7f7a)

![image.png](https://note.youdao.com/yws/res/21063/WEBRESOURCE4bdaf306ab2b25192fbf4686f44aacd4)


## 并发服务器模型

### 实例

#### 预先创建一定量的进程等待客户端的请求

```c
for(int i=0;i<4;i++){
    fork()
}
```

创建 `16`个进程（`2^4`）并发执行。


> server.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>

#define PORT_ID	8800
#define SIZE	100

int main(void)
{
	int i;

	int sockfd, client_sockfd;
	struct sockaddr_in my_addr, client_addr;
	int addr_len;
	char welcome[SIZE] = "Welcome to connect to the sever!";

	//1.socket()
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.bind()
	my_addr.sin_family = AF_INET;
	my_addr.sin_port = htons(PORT_ID);
	my_addr.sin_addr.s_addr = INADDR_ANY;
	bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr));

	//3.listen()
	listen(sockfd, 10);

	addr_len = sizeof(struct sockaddr);


	for(i = 0; i < 3; i++)
		fork();

	while(1)
	{
		printf("Server is waiting for client to connect:\n");
		//4.accept()
		client_sockfd = accept(sockfd, (struct sockaddr *)&client_addr, &addr_len);
		printf("Client IP address = %s\n", inet_ntoa(client_addr.sin_addr));
		//5.send()
		send(client_sockfd, welcome, SIZE, 0);
		printf("Disconnect the client request.\n");
		//6.close()
		close(client_sockfd);
	}

	close(sockfd);

	return 0;
}
```

客户端代码保持不变

> clinet.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>

#define PORT_ID	8800
#define SIZE	100

//./client IP  : ./client 192.168.0.10
int main(int argc, char *argv[])
{
	int sockfd;
	struct sockaddr_in server_addr;
	char buf[SIZE];

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket()
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.connect()
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT_ID);
	server_addr.sin_addr.s_addr = inet_addr(argv[1]);
	connect(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));

	recv(sockfd, buf, SIZE, 0);
	printf("Client receive from server: %s\n", buf);

	close(sockfd);

	return 0;
}
```


- 运行结果：

![image.png](https://note.youdao.com/yws/res/21064/WEBRESOURCEa91cd127683e645ca0fc0e597d1c312e)

![image.png](https://note.youdao.com/yws/res/21065/WEBRESOURCE1b7f0689696c36cce4e04e69895a73b8)


#### 当客户端请求到来时，才创建处理客户端请求的线程

> server.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <pthread.h>

#define PORT_ID	8800
#define SIZE	100

void *thread_function(void *arg);
struct sockaddr_in client_addr;
int client_sockfd;
char welcome[SIZE] = "Welcome to connect to the sever!";

int main(void)
{
	int sockfd;
	struct sockaddr_in my_addr;
	int addr_len;
	pthread_t pthread;

	//1.socket()
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.bind()
	my_addr.sin_family = AF_INET;
	my_addr.sin_port = htons(PORT_ID);
	my_addr.sin_addr.s_addr = INADDR_ANY;
	bind(sockfd, (struct sockaddr *)&my_addr, sizeof(struct sockaddr));

	//3.listen()
	listen(sockfd, 10);

	addr_len = sizeof(struct sockaddr);

	while(1)
	{
		printf("Server is waiting for client to connect:\n");
		client_sockfd = accept(sockfd, (struct sockaddr *)&client_addr, &addr_len);
		pthread_create(&pthread, NULL, thread_function, NULL);
	}

	close(sockfd);

	return 0;
}

void *thread_function(void *arg)
{
	printf("Client IP address = %s\n", inet_ntoa(client_addr.sin_addr));
	send(client_sockfd, welcome, SIZE, 0);
	printf("Disconnect the client request.\n");
	close(client_sockfd);
	pthread_exit(NULL);

	return NULL;
}
```

> client.c

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include <unistd.h>
#include <stdlib.h>

#define PORT_ID	8800
#define SIZE	100

//./client IP  : ./client 192.168.0.10
int main(int argc, char *argv[])
{
	int sockfd;
	struct sockaddr_in server_addr;
	char buf[SIZE];

	if(argc < 2)
	{
		printf("Usage: ./client [server IP address]\n");
		exit(1);
	}

	//1.socket()
	sockfd = socket(AF_INET, SOCK_STREAM, 0);

	//2.connect()
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT_ID);
	server_addr.sin_addr.s_addr = inet_addr(argv[1]);
	connect(sockfd, (struct sockaddr *)&server_addr, sizeof(struct sockaddr));

	recv(sockfd, buf, SIZE, 0);
	printf("Client receive from server: %s\n", buf);

	close(sockfd);

	return 0;
}
```

- 运行结果：

![image.png](https://note.youdao.com/yws/res/21066/WEBRESOURCEde6a63185d0448ddbc12ed345a8accbb)

![image.png](https://note.youdao.com/yws/res/21067/WEBRESOURCEefa201ecf8575b219288c4a18abc8bad)


