---
layout: post
title: "�ں�ѧϰ-���̹���"
description: "�ں�ѧϰ ���̹���"
category: kernel
tags: [process]
---

##1. ���̹���

###1.1 ���̸���
���̾��Ǵ���ִ���ڵĳ����ǳ���ִ�е�һ��ʵ����Ҫ���ǡ����򡱼ӡ�ִ�С��������ĳ����Ǵ����ĳ�ֽ����еĶ����ƴ��롣
���̴�����ʱ�򣬼����븸������ͬ���븸��������ͬ�ĳ�����룬�����и��Զ��������ݿ��������Ѻ�ջ���ӽ��̶�һ���ڴ浥Ԫ���޸ģ��������ǲ��ɼ��ģ���֮��Ȼ��

###1.2 �̸߳���
������������̿��Թ���һЩ��Դ�������ֿ������ں˶������ȣ�һ��˯����һ���ǿ������еġ����������̺��̹߳������������̵߳ĸ��

###1.3����������&�߳�������
include/linux/sched.h��

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

�����������а����������ܹ�����������һ������ִ�еĳ��򣬺����˽��̴򿪵��ļ������̵ĵ�ַ�ռ䣬������źţ����̵�״̬�ȵ���Ϣ��

###1.4 ���̶�ջ
һ��������������ջ���û���ջ���ں˶�ջ
����ִ��ϵͳ���������ں˺󣬴�����ת��Ϊ����Ȩģʽ������ARM��˵��ͨģʽ���û�ģʽ��ջ�루SP���ǲ�ͬ�ļĴ�������ʱʹ�õ�ջָ������ں�ջָ�룬��ָ���ں�Ϊÿ�����̷�����ں�ջ�ռ䡣�ں�ջͬʱ���ڱ���һЩϵͳ����ǰ��Ӧ�ò���Ϣ�����û��ռ�ջָ�롢ϵͳ���ò�������

Linux��������ͬ�����ݽṹ���յķ���һ������Ϊ���̷���Ĵ洢�����ڡ�һ�����ں�̬�Ľ��̶�ջ����һ�����߳�������thread_info�ṹ��

include/linux/sched.h��
```
union thread_union {
	struct thread_info thread_info;	//�߳�������
	unsigned long stack[THREAD_SIZE/sizeof(long)];	//�ں�̬�Ľ��̶�ջ
};
```

arch/arm/include/asm/thread_info.h��
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


�öδ洢�����ʾ��ͼΪ��

![Alt text](public/img/posts/process0.png)


`#define THREAD_SIZE		8192(0x2000)`

Arm��sp�Ĵ�����CPU��ջָ�룬�������ջ����Ԫ�ĵ�ַ�����û�̬�ո��л����ں�̬��ʱ�򣬽��̵��ں�ջ���ǿյģ�ջ��ʼ������ڴ�����ĩ�ˣ�������ʼ�ķ������������ͨ����ǰ��ջ��sp��ֵ��õ�ǰ����ڴ�������ʼ��ַ��������Ĵ��룺

```
static inline struct thread_info *current_thread_info(void)
{
register unsigned long sp asm (��sp��);
return (struct thread_info *)(sp & ~(THREAD_SIZE -1));
}
```

ͨ������Ĵ��������ͨ��sp���thread_info���ڴ��е���ʼ��ַ�����̳��õ��ǽ�����������ַ������thread_info������Ϊ�˻���ڵ�ǰCPU�����е�������ָ�룬����ʹ�ã���Linux��current.h�������code��

```
#define get_current() (current_thread_info()->task)
#define current get_current()
```

���Խ���ͨ������ں�ջ�����ܻ�õ�ǰ��ȷ�Ľ��̡�


###1.5 ���̹�ϵ
���򴴽����̾��и��ӹ�ϵ�����һ�����̴�������ӽ��̣����ӽ���֮������ֵܹ�ϵ��

������һ������p��������task_struct���������ֶΣ�

Real_parent��ָ���˴�������p�ĸ����̵���������

Parent��ָ����p�ĵ�ǰ�ĸ����̡�

Children�������ͷ��������������Ԫ�ض���p�������ӽ��̡�

Sibling��ָ���ֵܽ����У���һ������ǰһ��siblingԪ�ص�ָ�롣

###1.6����״̬
���̵�״̬�����Ϸ�Ϊ

��1���������л�����������

��2�����̹���interruptible��uninterruptible��

��3������ֹͣ����ͣ��

��4�����̱����٣���ͣ

include/linux/sched.h�ں��еĺ궨�壺

```
#define TASK_RUNNING		0		//Ҫô��CPU�����У�Ҫô׼������
#define TASK_INTERRUPTIBLE	1		//���̱�����ֱ��Ӳ���ж��ͷŽ��̵ȴ�����Դ�����߲���һ���źŶ����԰ѽ���״̬����ΪTASK_RUNNING��
#define TASK_UNINTERRUPTIBLE	2	//ͬ�ϣ������źŲ��ܻ�������
#define __TASK_STOPPED		4		//���̵�ִ�б���ͣ��
#define __TASK_TRACED		8		//���̵�ִ�б�debugger��ͣ��
```

###1.7���̶���
###1.7.1�������
ͨ��˫��������ں��еĽ�����ϵ������
�������еĽ��̣�
```
#define for_each_process(p) \
	for (p = &init_task ; (p = next_task(p)) != &init_task ; )
```

###1.7.2���ж���
���д���TASK_RUNNING״̬�Ľ�����ɵĶ��С����̵����ж����Ǹ�����ĸ������CFS��ƽ�����㷨ʹ�õ����ж���ʹ�ú��������֯�ġ������㷨ͨ��ĳ�ֵ��Ȳ�����ʵ�ֶԿ����ж��е���������

###1.7.3�ȴ�����
���д���TASK_UNINTERRUPTIBLE��TASK_INTERRUPTIBLE״̬�Ľ�����ɵĶ��С�
������ȴ����к��ź������ʹ�õ�ʱ�򣬽���������


![Alt text](public/img/posts/process1.png)


���Ǿ�����д������ʱ���ڵȴ�ĳ��������ʱ����Ҫ����һ�����̣����Ǿ���ʹ��һЩLinux��API�����磺
include/linux/wait.h��
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

finish_wait�ѽ��̵�״̬�ٴ�����ΪTASK_RUNNING״̬���������ڵ���schedule()֮ǰ��������Ϊ�������¡�

DEFINE_WAIT(__wait); ��ʼ��һ����wait_queue_t�ĵȴ�����Ԫ�أ��õȴ����нṹԭ��Ϊ��

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

�ú����ѵ�ǰ���̵�task_structָ�븳ֵ��privateָ��������Ӷ�����ͼ��ʾ��һ�������ĵȴ�����Ԫ�س�ʼ����ϣ����Һ���Ӧ�Ľ��������������autoremove_wake_functionΪ�����������prepare_to_wait_event �������ʼ�����˵�__wait�ȴ�����Ԫ�أ���ӵ���wq��Ϊ�ȴ�����ͷ�Ľ��̵ȴ������С�Wq�ĳ�ʼ��Ϊ��
`DECLARE_WAIT_QUEUE_HEAD(wq)`

DECLARE_WAIT_QUEUE_HEAD�Ǹ��궨�壺

```
#define __WAIT_QUEUE_HEAD_INITIALIZER(name) {				\
	.lock		= __SPIN_LOCK_UNLOCKED(name.lock),		\
	.task_list	= { &(name).task_list, &(name).task_list } }

#define DECLARE_WAIT_QUEUE_HEAD(name) \
	wait_queue_head_t name = __WAIT_QUEUE_HEAD_INITIALIZER(name)
```

��ɵȴ�����ͷ�ĳ�ʼ����

```
void
prepare_to_wait(wait_queue_head_t *q, wait_queue_t *wait, int state)
{
	unsigned long flags;

	wait->flags &= ~WQ_FLAG_EXCLUSIVE;
ע�ͣ�ldd3������flagsΪ0��ʾ�ǻ�����̣���֪��Ϊʲô���ʼ�ն�������Ϊ0��
�ѵ�wait_event_xxx��ص�API���Ƿǻ�����̣����������ȴ������ٽ���Դ��
�����Ҵ���prepare_to_wait�������������ר�Ų���ǻ���ȴ����еġ�
prepare_to_wait_exclusive�����ǲ��뻥��ȴ����еġ�
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

###1.8���̴���
LinuxӦ�ó����У�clone()��vfork()��fork()��Ϊ���̴�����ϵͳ���á�

###1.9�ں˽���
###1.9.1 ����
��Linux�ں��У���ν���ں��߳�ʵ������һ���������̵�ַ�ռ�Ľ��̣������Լ���ϵͳ��ջ������������Ȼ��һ�����̣�ֻ������Щ���̿������������̹���ĳЩ��Դ���������������Ҳ����ν���̡߳�

###1.9.2����
�ں��߳�û���Լ��ĵ�ַ�ռ䣬�������ǵġ�current->mm�����ǿյģ�
�ں��߳�ֻ�����ں˿ռ�������������û��ռ佻����
����ͨ����һ�����ں��߳�Ҳ�����ȼ��ͱ����ȡ�

###1.9.3����
kthread_create�ӿڣ����Ǳ�׼���ں��̴߳����ӿڣ�ֻ����øýӿڱ�ɴ����ں��̣߳�
Ĭ�ϴ������߳��Ǵ��ڲ������е�״̬��������Ҫ�ڸ�������ͨ������wake_up_process()�������������̡߳�����һ���ں��̲߳�Ҫ���������������Ե���kthread_run�ӿں�����

���ǻ�ע�⵽kernel_thread����Ҳ�ǿ��Դ����ں��̵߳ġ������������ں˵�һ���߳�1.

###1.9.4�˳�
���߳�ִ�е�����ĩβʱ���Զ������ں���do_exit()�������˳��������̵߳���kthread_stop()��ָ���߳��˳�

###1.10����0
###1.10.1����0
���н��̵����Ƚ�������0��Idle���̣�����Linux�ĳ�ʼ���׶δ��޵��д�����һ���ں��̡߳�

init/init_task.c��

`struct task_struct init_task = INIT_TASK(init_task);`

###1.10.2�߳�1
Start_kernel������ʼ���ں���Ҫ���������ݽṹ�������жϣ�������һ���ں��߳�1��
kernel_init��

`kernel_thread(kernel_init, NULL, CLONE_FS | CLONE_SIGHAND);`

�����ں��߳�1�����0����idle״̬��

`cpu_idle();`

###1.10.3����1
�߳�1 ��kernel_init���ڽ������ں˳�ʼ���󣬻���ã�

`run_init_process(��/sbin/init��);`

�ú�����Ҫ���þ���ͨ��execve()ϵͳ����װ���ִ�г���init���Ӷ��ں��߳�1�����һ����ͨ�Ľ���1��