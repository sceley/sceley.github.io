---
title: C语言中的多进程
date: 2018-06-04 10:03:26
tags: C语言
---

## 序言

为了充分利用计算机中的多核CPU，计算机提供了两个接口使用多核CPU，两个接口分别是：多进程、多线程。本编文章将介绍多进程编程，利用多核CPU。

<!-- more -->

## 多进程

> 核的个数与可同时运行的进程数相同。相反，若进程数超过核数，进程将分时使用CPU资源。但因为CPU运行速度极快，我们会感到所有进程同时运行。当然多核越多，这种感觉也明显。

### fork函数创建多进程

```c
#include <unistd.h>

pid_t fork(void);
```
fork函数将创建调用的进程副本。两个进程都将执行fork调用后的语句，子进程将复制父进程相同的内存空间，之后的程序流要根据fork函数的返回值加以区分。

- 父进程：fork函数返回子进程ID
- 子进程：fork函数返回0

![](/img/fork函数创建进程.png)

eg.

```c
#include <unistd.h>
#include <stdio.h>

int gval = 10;
int main () {
	pid_t pid;
	int lval = 20;
	gval++, lval += 5;
	
	pid = fork();
	if (pid == 0) {//child pro
		gval += 2, lval += 2;
	} else {//parent pro
		gval -= 2, lval -= 2;
	}
	
	if (pid == 0) {
		printf("Child Proc: [%d, %d] \n", gval, lval);
	} else {
		printf("Parent Proc: [%d, %d] \n", gval, lval);
	}
}
```
运行结果：

![](/img/fork.png)

### 进程与僵尸进程

#### 僵尸进程的产生

文件操作中，关闭文件和打开文件同等重要。同样进程的创建和进程的销毁同等重要。如果未认真对待进程销毁，它们将变成僵尸进程困扰各位。

进程完成工作后（执行完main函数中的程序后）应当销毁，但有时这些进程将变成僵尸进程，占用系统中的重要资源。

子进程的终止方式：

- 传递参数并调用exit函数
- main函数中执行return语句并返回值

向exit函数传递的参数值和main函数的return语句返回的值都会传递给操作系统。而操作系统并不会销毁子进程，直到把这些值传递给产生该子进程的父进程。

eg.

```c
#include <stdio.h>
#include <unistd.h>

int main () {
	pid_t pid = fork();
	if (pid == 0) { Child Pro
		printf("Hi, I am a child process \n");
	} else {
		printf("Child Process ID: %d \n", pid);
	}
	
	if (pid) {
		puts("End child process \n");
	} else {
		printf("End parent process \n");
	}
}
```

#### 销毁僵尸进程

为了销毁子进程，父进程应主动请求获取子进程的返回值。

##### 使用wait函数

```c
#include <sys/wait.h>

pid wait(int * statloc);
//成功时返回终止的子进程的ID，失败时返回-1
```
调用此函数时如果已有子进程终止，那么子进程终止时传递的返回值（exit函数的参数值、main函数的return返回值）将保存到该函数的参数所指内存空间。

- WIFEXITED子进程正常终止时返回”真“ true
- WEXITSTATUS返回子进程的返回值

eg.

```c
#include <stdio.h>
#in
#include <unistd.h>

int main () {
	int status;
	pid_t pid = fork();
	if (pid == 0) {
		return 3;
	} else {
		printf("Child PID:%d \n", pid);
		pid = fork();
		if (pid == 0) {
			exit(7);
		} else {
			wait(&status);
			if (WIFEXITED(status)) {
				printf("Child send one: %d \n", WEXITSTATUS(status));
			}
			wait(&status)
			if (WIFEXITED(status)) {
				printf("Child send tow: %d \n", WEXITSTATUS(status));
			}
			sleep(30);
		}
	}
}
```

运行结果：

![](/img/wait.png)

调用wait函数时，如果没有已终止的子进程，那么程序将阻塞（Blocking）直到有子进程终止。

##### 使用waitpid函数

wait函数会引起程序阻塞，而waitpid即使没有终止的子进程也不会进入阻塞状态，而是返回0并退出。

```c
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int * statloc, int options);

//成功时返回终止的子进程ID(或0)，失败时返回-1

//1. pid 等待终止的目标子进程的ID， 若传递-1，则与wait函数相同，可以等待任意子进程终止
//2. ...
//3. 传递头文件sys/wait.h中声明的常量WNOHANG，即使没有终止的子进程也不会进入阻塞状态，而是返回0并退出
```

调用waitpid时程序不会阻塞。

eg.

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main () {
	int status;
	pid_t pid = fork();
	
	if (pid == 0) {
		return 24;
	} else {
		while (!waitpid(-1, &status, WNOHANG)) {
			sleep(3);
			printf("sleep 3sec \n");
		}
		if (WIFEXITED(status)) {
			printf("Child send %d \n", WEXITSTATUS(status));
		}
	}
	return 0;
}
```

运行结果：

![](/img/waitpid.h)

### 信号处理

#### signal函数

信号注册函数

```c
#include <signal.h>

void (*signal(int signo, void (*func)(int)))(int);

//函数名: signal
//参数：int signo, void (*func)(int)
//返回类型：参数为int类型，返回void型函数指针
```

发生第一个参数代表的情况时，调用第二个参数所指向的函数。

signal函数中注册的部分特殊情况和对应的常数。

- SIGALRM: 已通过调用alarm函数注册的时间
- SIGINT: 输入CTRL+C
- SIGCHLD: 子进程终止

注册好信号后，发生注册信号时（注册的情况发生时），操作系统将调用该信号对应的函数。

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
```

如果调用该函数的同时向它传递一个正整型参数，相应时间后（以秒为单位）将产生SIGALRM信号。若向该函数传递0，则之前对SIGALRM信号的预约将取消。

eg.

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void timeout(int sig) {
	if (sig == SIGALRM) {
		puts("Time out!");
	}
	alarm(2);
};

void keycontrol(int sig) {
	if (sig == SIGINT) {
		puts("CTRL+C");
	}
}

int main () {
	int i;
	signal(SIGALRM, timeout);
	signal(SIGINT, keycontrol);
	alarm(2);
	
	for (int i = 0; i < 3; i++) {
		puts("waiting ...");
	}
	return 0;
}

```

运行结果：

![](/img/signal.png)

> 发生信号时将唤醒由于调用sleep函数而进入阻塞状态的进程

#### sigaction函数进行信号处理

```c
#include <signal.h>

int sigaction (int signo, const struct sigaction *act, struct sigaction *oldact);
```