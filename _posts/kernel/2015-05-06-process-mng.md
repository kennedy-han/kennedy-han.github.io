---
layout: post
title: "内核学习 进程管理"
description: "内核学习 进程管理"
category: kernel
tags: [kernel]
---

##1. 进程管理

###1.1 进程概念
进程就是处于执行期的程序，是程序执行的一个实例，要素是“程序”加“执行”。单纯的程序是存放在某种介质中的二进制代码。
进程创建的时候，几乎与父进程相同，与父进程有相同的程序代码，但是有各自独立的数据拷贝——堆和栈。子进程对一个内存单元的修改，父进程是不可见的，反之亦然。

###1.2 线程概念
多个轻量级进程可以共享一些资源，但是又可以由内核独立调度，一个睡眠另一个是可以运行的。轻量级进程和线程关联起来就是线程的概念。

###1.3进程描述符&线程描述符
include/linux/sched.h：

```
struct task_struct {
	volatile long state;	/* -1 unrunnable, 0 runnable, >0 stopped */
	void *stack;
	atomic_t usage;
	unsigned int flags;	/* per process flags, defined below */
	unsigned int ptrace;
.............................
.............................
.............................
}
```

进程描述符中包含的数据能够完整的描述一个正在执行的程序，涵盖了进程打开的文件，进程的地址空间，挂起的信号，进程的状态等等信息。

###1.4 进程堆栈
一个进程有两个堆栈：用户堆栈，内核堆栈
进程执行系统调用陷入内核后，处理器转换为了特权模式，对于ARM来说普通模式和用户模式的栈针（SP）是不同的寄存器，此时使用的栈指针就是内核栈指针，他指向内核为每个进程分配的内核栈空间。内核栈同时用于保存一些系统调用前的应用层信息（如用户空间栈指针、系统调用参数）。

Linux把两个不同的数据结构紧凑的放在一个单独为进程分配的存储区域内。一个是内核态的进程堆栈，另一个是线程描述符thread_info结构。

include/linux/sched.h：
```
union thread_union {
	struct thread_info thread_info;	//线程描述符
	unsigned long stack[THREAD_SIZE/sizeof(long)];	//内核态的进程堆栈
};
```

arch/arm/include/asm/thread_info.h：
```
struct thread_info {
	unsigned long		flags;		/* low level flags */
	int			preempt_count;	/* 0 => preemptable, <0 => bug */
	mm_segment_t		addr_limit;	/* address limit */
	struct task_struct	*task;		/* main task structure */
	struct exec_domain	*exec_domain;	/* execution domain */
	__u32			cpu;		/* cpu */
	__u32			cpu_domain;	/* cpu domain */
	struct cpu_context_save	cpu_context;	/* cpu context */
	__u32			syscall;	/* syscall number */
	__u8			used_cp[16];	/* thread used copro */
	unsigned long		tp_value[2];	/* TLS registers */
#ifdef CONFIG_CRUNCH
	struct crunch_state	crunchstate;
#endif
	union fp_state		fpstate __attribute__((aligned(8)));
	union vfp_state		vfpstate;
#ifdef CONFIG_ARM_THUMBEE
	unsigned long		thumbee_state;	/* ThumbEE Handler Base register */
#endif
	struct restart_block	restart_block;
};
```


该段存储区域的示意图为：

![Alt text](public/img/posts/process0.png)


`#define THREAD_SIZE		8192(0x2000)`

Arm的sp寄存器是CPU的栈指针，用来存放栈顶单元的地址。从用户态刚刚切换到内核态的时候，进程的内核栈总是空的，栈起始与这段内存区的末端，并朝开始的方向增长。如何通过当前堆栈的sp的值获得当前这段内存区的起始地址，见下面的代码：

```
static inline struct thread_info *current_thread_info(void)
{
register unsigned long sp asm (“sp”);
return (struct thread_info *)(sp & ~(THREAD_SIZE -1));
}
```

通过上面的代码很容易通过sp获得thread_info在内存中的起始地址。进程常用的是进程描述符地址而不是thread_info，所以为了获得在当前CPU上运行的描述符指针，可以使用：在Linux的current.h中有这段code：

```
#define get_current() (current_thread_info()->task)
#define current get_current()
```

所以仅仅通过检查内核栈，就能获得当前正确的进程。


###1.5 进程关系
程序创建进程具有父子关系，如果一个进程创建多个子进程，则子进程之间具有兄弟关系。

假设有一个进程p，则他的task_struct中有如下字段：

Real_parent：指向了创建进程p的父进程的描述符。

Parent：指向了p的当前的父进程。

Children：链表的头部，链表中所有元素都是p创建的子进程。

Sibling：指向兄弟进程中，下一个或者前一个sibling元素的指针。

###1.6进程状态
进程的状态大体上分为

（1）正在运行或者马上运行

（2）进程挂起（interruptible，uninterruptible）

（3）进程停止（暂停）

（4）进程被跟踪，暂停

include/linux/sched.h内核中的宏定义：

```
#define TASK_RUNNING		0		//要么在CPU上运行，要么准备运行
#define TASK_INTERRUPTIBLE	1		//进程被挂起，直到硬件中断释放进程等待的资源，或者产生一个信号都可以把进程状态设置为TASK_RUNNING。
#define TASK_UNINTERRUPTIBLE	2	//同上，但是信号不能唤醒他。
#define __TASK_STOPPED		4		//进程的执行被暂停。
#define __TASK_TRACED		8		//进程的执行被debugger暂停。
```

###1.7进程队列
###1.7.1任务队列
通过双向链表把内核中的进程联系起来。
遍历所有的进程：
```
#define for_each_process(p) \
	for (p = &init_task ; (p = next_task(p)) != &init_task ; )
```

###1.7.2运行队列
所有处于TASK_RUNNING状态的进程组成的队列。进程的运行队列是个抽象的概念，例如CFS公平调度算法使用的运行队列使用红黑树来组织的。调度算法通过某种调度策略来实现对可运行队列的增减任务。

###1.7.3等待队列
所有处于TASK_UNINTERRUPTIBLE，TASK_INTERRUPTIBLE状态的进程组成的队列。
当多个等待队列和信号量混合使用的时候，谨防死锁。


![Alt text](public/img/posts/process1.png)


我们经常在写驱动的时候在等待某个条件的时候需要阻塞一个进程，我们经常使用一些Linux的API，比如：
include/linux/wait.h：
```
#define wait_event(wq, condition)					\
do {									\
	might_sleep();							\
	if (condition)							\
		break;							\
	__wait_event(wq, condition);					\
} while (0)
```

```
#define ___wait_event(wq, condition, state, exclusive, ret, cmd)	\
({									\
	__label__ __out;						\
	wait_queue_t __wait;						\
	long __ret = ret;	/* explicit shadow */			\
									\
	INIT_LIST_HEAD(&__wait.task_list);				\
	if (exclusive)							\
		__wait.flags = WQ_FLAG_EXCLUSIVE;			\
	else								\
		__wait.flags = 0;					\
									\
	for (;;) {							\
		long __int = prepare_to_wait_event(&wq, &__wait, state);\
									\
		if (condition)						\
			break;						\
									\
		if (___wait_is_interruptible(state) && __int) {		\
			__ret = __int;					\
			if (exclusive) {				\
				abort_exclusive_wait(&wq, &__wait,	\
						     state, NULL);	\
				goto __out;				\
			}						\
			break;						\
		}							\
									\
		cmd;							\
	}								\
	finish_wait(&wq, &__wait);					\
__out:	__ret;								\
})
```

```
#define __wait_event(wq, condition)					\
	(void)___wait_event(wq, condition, TASK_UNINTERRUPTIBLE, 0, 0,	\
			    schedule())
```

finish_wait把进程的状态再次设置为TASK_RUNNING状态，仅发生在调用schedule()之前唤醒条件为真的情况下。

DEFINE_WAIT(__wait); 初始化一个叫wait_queue_t的等待队列元素，该等待队列结构原型为：

```
struct __wait_queue {
	unsigned int		flags;
	void			*private;
	wait_queue_func_t	func;
	struct list_head	task_list;
};
```

```
#define DEFINE_WAIT_FUNC(name, function)				\
	wait_queue_t name = {						\
		.private	= current,				\
		.func		= function,				\
		.task_list	= LIST_HEAD_INIT((name).task_list),	\
	}

#define DEFINE_WAIT(name) DEFINE_WAIT_FUNC(name, autoremove_wake_function)
```

该函数把当前进程的task_struct指针赋值给private指针变量，从而如上图所示，一个完整的等待队列元素初始化完毕，并且和相应的进程相关联起来。autoremove_wake_function为下面这个函数prepare_to_wait_event 把上面初始化好了的__wait等待队列元素，添加到以wq作为等待队列头的进程等待队列中。Wq的初始化为：
`DECLARE_WAIT_QUEUE_HEAD(wq)`

DECLARE_WAIT_QUEUE_HEAD是个宏定义：

```
#define __WAIT_QUEUE_HEAD_INITIALIZER(name) {				\
	.lock		= __SPIN_LOCK_UNLOCKED(name.lock),		\
	.task_list	= { &(name).task_list, &(name).task_list } }

#define DECLARE_WAIT_QUEUE_HEAD(name) \
	wait_queue_head_t name = __WAIT_QUEUE_HEAD_INITIALIZER(name)
```

完成等待队列头的初始化。

```
void
prepare_to_wait(wait_queue_head_t *q, wait_queue_t *wait, int state)
{
	unsigned long flags;

	wait->flags &= ~WQ_FLAG_EXCLUSIVE;
注释：ldd3中描述flags为0表示非互斥进程，不知道为什么这边始终都是设置为0。
难道wait_event_xxx相关的API都是非互斥进程？不能用来等待访问临界资源。
最后查找代码prepare_to_wait发现这个函数是专门插入非互斥等待队列的。
prepare_to_wait_exclusive函数是插入互斥等待队列的。
	spin_lock_irqsave(&q->lock, flags);
	if (list_empty(&wait->task_list))
		__add_wait_queue(q, wait);
	set_current_state(state);
	spin_unlock_irqrestore(&q->lock, flags);
}
EXPORT_SYMBOL(prepare_to_wait);
```

```
struct __wait_queue_head {
	spinlock_t		lock;
	struct list_head	task_list;
};
typedef struct __wait_queue_head wait_queue_head_t;
```

```
static inline void __add_wait_queue(wait_queue_head_t *head, wait_queue_t *new)
{
	list_add(&new->task_list, &head->task_list);
}
```

###1.8进程创建
Linux应用程序中，clone()，vfork()，fork()的为进程创建的系统调用。

###1.9内核进程
###1.9.1 定义
在Linux内核中，所谓的内核线程实际上是一个共享父进程地址空间的进程，它有自己的系统堆栈；所以他们依然是一个进程，只不过这些进程可以与其他进程共享某些资源，这里的其他进程也是所谓的线程。

###1.9.2区别
内核线程没有自己的地址空间，所以他们的”current->mm”都是空的；
内核线程只能在内核空间操作，不能与用户空间交互；
跟普通进程一样，内核线程也有优先级和被调度。

###1.9.3创建
kthread_create接口，则是标准的内核线程创建接口，只须调用该接口便可创建内核线程；
默认创建的线程是存于不可运行的状态，所以需要在父进程中通过调用wake_up_process()函数来启动该线程。创建一个内核线程并要它运行起来，可以调用kthread_run接口函数。

我们还注意到kernel_thread函数也是可以创建内核线程的。由他来创建内核第一个线程1.

###1.9.4退出
当线程执行到函数末尾时会自动调用内核中do_exit()函数来退出或其他线程调用kthread_stop()来指定线程退出

###1.10进程0
###1.10.1进程0
所有进程的祖先叫做进程0，Idle进程，他是Linux的初始化阶段从无到有创建的一个内核线程。

init/init_task.c：

`struct task_struct init_task = INIT_TASK(init_task);`

###1.10.2线程1
Start_kernel函数初始化内核需要的所有数据结构，激活中断，创建另一个内核线程1：
kernel_init。

`kernel_thread(kernel_init, NULL, CLONE_FS | CLONE_SIGHAND);`

创建内核线程1后进程0进入idle状态：

`cpu_idle();`

###1.10.3进程1
线程1 ‘kernel_init’在进行了内核初始化后，会调用：

`run_init_process(“/sbin/init”);`

该函数主要作用就是通过execve()系统调用装入可执行程序init，从而内核线程1变成了一个普通的进程1。