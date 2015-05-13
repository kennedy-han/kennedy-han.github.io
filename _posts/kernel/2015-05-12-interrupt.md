---
layout: post
title: "内核学习-->中断机制"
description: "内核 中断"
category: kernel
tags: [kernel]
---

##3.中断机制

###3.1数据结构

三个数据结构：

中断全局描述符：irq_desc

中断服务描述符：irqaction

中断硬件描述符：irq_chip

![Alt text](/public/img/posts/interrupt0.png)

###3.1.1中断描述符结构irq_desc

irq_desc：中断全局描述符

```
struct irq_desc {
	struct irq_data		irq_data;		注释：表示此中断描述符的中断号
	unsigned int __percpu	*kstat_irqs;
	irq_flow_handler_t	handle_irq;	注释：当前中断的内核中断处理函数入口（非用户定义）
#ifdef CONFIG_IRQ_PREFLOW_FASTEOI
	irq_preflow_handler_t	preflow_handler;
#endif
	struct irqaction	*action;	/* IRQ action list */
注释：标识当出现IRQ时要调用的中断服务例程。该字段指向IRQ的irqaction链表的第一个元素。
	unsigned int		status_use_accessors;
	unsigned int		core_internal_state__do_not_mess_with_it;
	unsigned int		depth;		/* nested irq disables */
注释：如果IRQ线被激活，则显示0，如果IRQ线被禁止了不止一次，则显示一个正数。
	unsigned int		wake_depth;	/* nested wake enables */
	unsigned int		irq_count;	/* For detecting broken IRQs */
	unsigned long		last_unhandled;	/* Aging timer for unhandled count */
	unsigned int		irqs_unhandled;
	atomic_t		threads_handled;
	int			threads_handled_last;
	raw_spinlock_t		lock;
	struct cpumask		*percpu_enabled;
#ifdef CONFIG_SMP
	const struct cpumask	*affinity_hint;
	struct irq_affinity_notify *affinity_notify;
#ifdef CONFIG_GENERIC_PENDING_IRQ
	cpumask_var_t		pending_mask;
#endif
#endif
	unsigned long		threads_oneshot;
	atomic_t		threads_active;
	wait_queue_head_t       wait_for_threads;
#ifdef CONFIG_PM_SLEEP
	unsigned int		nr_actions;
	unsigned int		no_suspend_depth;
	unsigned int		force_resume_depth;
#endif
#ifdef CONFIG_PROC_FS
	struct proc_dir_entry	*dir;
#endif
	int			parent_irq;
	struct module		*owner;
	const char		*name;
} ____cacheline_internodealigned_in_smp;
```

Linux内核将所有的中断统一编号，使用一个irq_desc结构数组来描述这些中断；每个数组项对应一个中断，也可能是一组中断，它们共用相同的中断号，里面记录了中断的名称、中断状态、中断标记（比如中断类型、是否共享中断等），提供了中断的底层硬件访问函数（清除、屏蔽、使能中断），提供了这个中断的处理函数入口，通过它可以调用用户注册的中断处理函数。

###3.1.2中断服务描述符irqaction

irqaction:每中断服务描述符

```
struct irqaction {
	irq_handler_t		handler;	注释：用户定义的当前中断的驱动中断处理函数
	void			*dev_id;
	void __percpu		*percpu_dev_id;
	struct irqaction	*next;
	irq_handler_t		thread_fn;
	struct task_struct	*thread;
	unsigned int		irq;
	unsigned int		flags;	注释：中断标识
	unsigned long		thread_flags;
	unsigned long		thread_mask;
	const char		*name;
	struct proc_dir_entry	*dir;
} ____cacheline_internodealigned_in_smp;
```

irq_desc 结构中的irqaction结构类型在include/linux/interrupt,h中定义。用户注册的每个中断处理函数用一个irqaction结构来表示，一个中断比如共享中断可以有多个处理函数，他们的irqaction结构链接成一个链表，以action为表头。他是在用户request_irq的时候第一次创建并被初始化。

###3.1.3 中断硬件描述符irq_chip

irq_chip结构类型是在include/linux/irq.h中定义，其中的成员大多用于操作底层硬件，比如设置寄存器以屏蔽中断，使能中断，清除中断等。所以称他为中断硬件描述符，主要是提供硬件操作的API。

irq_chip:

```
struct irq_chip {
	const char	*name;	注释：定义名字，通过/proc/interrupts可以查看这个名字
	unsigned int	(*irq_startup)(struct irq_data *data);
	void		(*irq_shutdown)(struct irq_data *data);
	void		(*irq_enable)(struct irq_data *data);	注释：使能IRQ，默认使能
	void		(*irq_disable)(struct irq_data *data);

	void		(*irq_ack)(struct irq_data *data);	注释：irq的全局ack，需要根据硬件设置必须实现通常是清除当前中断使得可以接收下一个中断。
	void		(*irq_mask)(struct irq_data *data);		注释：屏蔽中断
	void		(*irq_mask_ack)(struct irq_data *data);
	void		(*irq_unmask)(struct irq_data *data);	注释：非屏蔽中断
	void		(*irq_eoi)(struct irq_data *data);

	int		(*irq_set_affinity)(struct irq_data *data, const struct cpumask *dest, bool force);
	int		(*irq_retrigger)(struct irq_data *data);
	int		(*irq_set_type)(struct irq_data *data, unsigned int flow_type);
	int		(*irq_set_wake)(struct irq_data *data, unsigned int on);

	void		(*irq_bus_lock)(struct irq_data *data);
	void		(*irq_bus_sync_unlock)(struct irq_data *data);

	void		(*irq_cpu_online)(struct irq_data *data);
	void		(*irq_cpu_offline)(struct irq_data *data);

	void		(*irq_suspend)(struct irq_data *data);
	void		(*irq_resume)(struct irq_data *data);
	void		(*irq_pm_shutdown)(struct irq_data *data);

	void		(*irq_calc_mask)(struct irq_data *data);

	void		(*irq_print_chip)(struct irq_data *data, struct seq_file *p);
	int		(*irq_request_resources)(struct irq_data *data);
	void		(*irq_release_resources)(struct irq_data *data);

	void		(*irq_compose_msi_msg)(struct irq_data *data, struct msi_msg *msg);
	void		(*irq_write_msi_msg)(struct irq_data *data, struct msi_msg *msg);

	unsigned long	flags;
};
```

`handle_edge_irq 与 handle_level_irq：`

根据代码可知，这两个函数的处理大同小异，都是调用handle_IRQ_event进行实质性的中断处理工作：他们的区别在于中断嵌套的处理。
* 在电平出发中断处理中，在handle_level_irq的一开始就调用了mask_ack_irq，屏蔽此中断，所以理论上不会产生同一个中断的嵌套调用。
* 在沿触发中断处理中，允许同一个中断的嵌套处理，此时使用IRQ_PENDING来标记嵌套，然后在handle_edge_irq中使用do-while来循环处理嵌套中断。沿触发中断在判断当前处于嵌套时，会调用mask禁止再一次出现嵌套，防止中断处理过程被频繁打断。

在实际写驱动过程中，经常会出现错误地使用handle_level_irq处理沿触发中断，或者错误地使用handle_edge_irq处理电平触发中断的情况，下面分析一下出现这两种错误时的现象：

**使用handle_level_irq处理沿触发中断**

由于handle_level_irq中调用了mask，直到中断处理完成之后才调用unmask，在这期间不会响应新的中断，所以沿触发的中断在这个过程中会丢失。

**使用handle_edge_irq处理电平触发中断**

handle_edge_irq不会调用mask，所以在调用handle_IRQ_event时，如果重新打开中断，就会进入嵌套。嵌套过程中会调用mask，禁止第二次嵌套出现，同时设置IRQ_PENDING标记。等到handle_IRQ_event返回时，由于那么IRQ_PENDING的存在，会再一次调用handle_IRQ_event，导致效率降低。

###3.2 request_irq
我们知道用户驱动程序通过`request_irq`函数向内核注册中断处理函数，request_irq函数根据中断号找到irq_desc数组项，然后在它的action链表添加一个表项。

```
error = request_irq(keypad->irq, keypad_ircLhandler,
            IRQF_DISABLED, pdev->name, keypad);
if (error) {
    deV_err(&pdev->dev, "failed to request IRQ\n");
    goto failed_put_clk;
}
```

看上面这段代码是我们在写驱动的时候需要申请IRQ号的时候需要写的一段标准code。
request_irq的原型为：

```
static inline int _must_check
request_irq(unsigned int irq, ircLhandler_t handler, unsigned long flags,
        const char *name, void *dev)
{
    return request_threaded_irq(irq, handler, NULL, flags, name, dev);
}
```

所以我们在写驱动的时候对应的各个参数：

keypad->irq 对应unsigned int irq ，中断号
keypad_irq_handler 对应 irq_handler_t handler，用户自定义的中断处理函数
IRQF_DISABLED 对应 unsigned long flags，Linux定义的中断flags如下：

```
define IRQF_DISABLED    0x00000020 注释：当前irq处理的时候保持irq禁止
define IRQF_SAMPLE_RANDOM 0x00000040
define IRQF_SHARED      0x00000080 注释：允许在多个设备之间共享irq号
define IRQF_PROBE_SHARED 0x00000100
define IRQF_TIMER       0x00000200 注释：标志该中断是timer interrupt
define IRQF_PERCPU      0x00000400
define IRQF_NOBALANCING 0x00000800
define IRQF_IRQPOLL     0x00001000
define IRQF ONESHOT     0x00002000
```

pdev->name 对应const char *name
keypad对应void *dev

`request_irq`调用了函数request_threaded_irq，他的原型为：

```
int request_threaded_irq(unsigned int irq, irq_hand1er_t hand1er,
            irq_hand1er_t thread_?1, unsigned long irqflags,
            const char *devname, void *dev_id)
{
    struct irqaction *action;
    struct ir(Ldesc *desc;
    ir1t retva1;
    
    /*
    * Sanity-check: shared interrupts must pass ni a real dev-ID,
    * otherwise We'll have trouble later trying to ?gure 0ut
    * Which interrupt is which (messes up the interrupt freeing
    * logic etc).
    */
    if ((irqflags & IRQF_SHARED) && !dev_id)
    retum -EINVAL;
    注释: 前面的 log 已经说的很明确 当多个 设备共享 irq 的时候, 我们必须通过 dev_id 来识别是哪一个设备来的中断, 所以当 flag 被标志为: IRQF_SHARED 共享的时候, 我们必须确保 dev_id 非空。
    
    desc = irq_to_desc(irq);    //的原型在下面 
    if (!desc)
        return -EINVAL; 注释: 如果没找到相应的irq_desc表示出错了。

    if (desc->status & IRQ_NOREQUEST)
        return -EINVAL;
注释: irq_desc.status初始化的时候设置为：IRQ_NOREQUEST不可被request状态，则这里做判断确保不能被request。
    
    if(!handler){
        if(!thread_fn)
            return -EINVAL;
        handler = irq_default_primary_handler;
    }
注释：如果在驱动中 request_irq 的时候没定义自己的 handler, 则至少 thread_fn 应该被定义
否则报错返回 EINVAL。 同时这段 code 还赋值一个某人 hand1er给驱动程序。

    action = kza11oc(sizeof(struct irqaction), GFP_KERNEL);
    if (!action)
    return -ENOMEM;
    
    action->hand1er = handler;
    action->thread_fn = thread_fn;
    action->flags = irqflags;
    action->name = devname;
    action->dev_id = dev_id;    

注释: kza11oc 一个 irqaction 结构体,并用驱动调用 request_irq 时候传进 去的 handler, irqflags ,devname 和 dev_id 初始化这个结构体

    chip_bus_1ock(irq, desc);    //函数原型在下面
    
    retva1 = __setup_irq(irq, desc, action);

注释: setup_irq 函数是是一个重要的函数, 将在下面进行具体讲解。

    chip_bus_sync_unlock(irq, desc);

    if (retva1)
        kfree(action);

#ifdef CONFIG_DEBUG_SHIRQ
if (!retval && (irqflags & IRQF_SHARED)) {
    /*
     * It's a shared IRQ -- the driver ought to be prepared for it
     * to happen immediate1y, so let's make sure....
     * We disable the irq to make sure that a 'real' IRQ doesn't
     * run in parallel with our fake.
     */
    unsigned long flags;
    
    disable_irq(irq);
    local_irq_save(flags);
    
    handler(irq, dev_id);
    
    local_irq_restore(flags);
    enable_irq(irq);

｝
#endif
    return retval;
}
EXPORT_SYMBOL(request_threaded_irq);
```

**irq_to_desc原型：**

```
struct irq_desc *irq_to_desc(unsigned int irq)
{
retum (irq < NR_IRQS) ? irq_desc + irq : NIJLL;
}
是通过 irq 号查找 irq_desc 数组, 来确定对应 irq 的 irq_desc[irq]

irq_desc 全局数组对应的定义在 kerne1/irq/hand1e.c 中, 如下:
struct irq_desc irq_desc〔NR_IRQS] __cacheline_a1igned_in_smp = {
    [0...NR_IRQS-1] = {
    .status = IRQ_DISABLED,
    .chip = &no_irq_chip,
    .handle_irq = handle_bad_irq,
    .depth = 1,
    .lock = __RAW_SPIN_LOCK_UNLOCKED(irq_desc->lock),
    }
};

NR_IRQS 一般会重定义在 arch/arm/mach-XXXX/inc1ude/mach/irqs.h

```

**chip_bus_1ock函数原型：**

```
/* Inline functions for support of irq chips on slow busses */
static inline void chip_bus_lock(unsigned int irq, struct irq_desc *desc)
{
if (un1ike1y(desc->chip->bus_lock))
    desc->chip->bus_lock(irq);
}

从linux给出的解释看出是为了在低速设备上的irq用的,就是实现了mutex。 具体实现是在 arch/arrn/mach-XXX/irq.c 中定义:
static struct irq_chip xxx_irq_chip = {
    .ack = xxx_irq_ack,
    .mask = xxx_irq_mask,
    .unmask = xxx_irq_unmask,
};
的时候有:
void (*bus_lock)(unsigned int irq);
void (*bus_sync_unlock)(unsigned int irq);
这两个函数的实现。
```

**__setup_irq函数：**

```
/*
* Interna1 function to register an irqaction - typically used to
* allocate special interrupts that are part of the architecture.
*/
static int
__setup_irq(unsigned int irq, struct irq_desc *desc, struct irqaction *new)
{
struct irqaction *old, **old_ptr;
const char *old_name = NULL;
unsigned long ?ags;
int nested, shared = 0;
int ret;

if(!desc)
    return -EINVAL;
if(desc->chip == &no_irq_chip)
    return -ENOSYS;
    
---
注释：这里判断chip是否被初始化，一般在arch/arm/mach-xxxx/irq.c的xxx_init_irq set_irq_chip(irqno,&xxx_irq_chip);中初始化这个desc->chip指针，xxx_irq_chip定义的数据结构类似：
    static  struct irq_chip xxx_irq_chip = {
        .ack = xxx_irq_ack;
        .mask = xxx_irq_mask;
        .unmask = xxx_irq_unmask;
    };    
---

/*
* Some drivers like serial.c use request_irq() heavily,
* so We have to be careful not to interfere With a
* running system.
*/
if (new->flags & IRQF_SAMPLE_RANDOM) {
    /*
    * This function might sleep, We want to call it first,
    * outside of the atomic block.
    * Yes, this might clear the entropy pool if the wrong
    * driver is attempted to be loaded without actually
    * installing a new handler, but is this really a problem,
    * only the sysadmin is able to do this.
    */
    rand_initia1ize_irq(irq);
}

/* Oneshot interrupts are not allowed with shared */
if((new->flags & IRQF_ONESHOT) && (new->flags & IRQF_SHARED))
    retum -EINVAL;
注释: Oneshot 类型的中断不允许共享

/*
* Check whether the interrupt nests into another interrupt
* thread.
*/
nested = desc->status & IRQ_NESTED_THREAD;
if (nested) {
    if (!new->thread_fn)
        return -EINVAL;
    /*
    * Replace file primary handler which was provided from
    * the driver for non nested interrupt handling by the
    * dummy function which warns when called.
    */
    new->handler = irq_nested_primary_handler;
}
```

继续上面的代码

![Alt text](/public/img/posts/interrupt1.png)

![Alt text](/public/img/posts/interrupt2.png)

![Alt text](/public/img/posts/interrupt3.png)

![Alt text](/public/img/posts/interrupt4.png)

![Alt text](/public/img/posts/interrupt5.png)

![Alt text](/public/img/posts/interrupt6.png)

![Alt text](/public/img/posts/interrupt7.png)

![Alt text](/public/img/posts/interrupt8.png)

![Alt text](/public/img/posts/interrupt9.png)

![Alt text](/public/img/posts/interrupt10.png)

![Alt text](/public/img/posts/interrupt11.png)

![Alt text](/public/img/posts/interrupt12.png)

------

###3.3 数据结构初始化
硬件相关的操作接口的实现是在arch/arm/mach-xxx/irq.c中进行，类似于下面的例子，他一般初始化irq_chip，irq_desc结构体的一些变量，并具体实现了操作硬件部分的函数。

![Alt text](/public/img/posts/interrupt13.png)

![Alt text](/public/img/posts/interrupt14.png)

![Alt text](/public/img/posts/interrupt15.png)

那么xxx_init_irq(void)函数什么时候调用呢？

这个还得看一个数据结构：

![Alt text](/public/img/posts/interrupt16.png)

![Alt text](/public/img/posts/interrupt17.png)

其中蓝色加粗函数就是定义了一个函数指针，一般来说我们在移植Linux内核到某款arm板的时候我们需要实现一个针对板子初始化的一个c文件mach-xxxboard.c，在这个c文件中我们需要实现以下code：

![Alt text](/public/img/posts/interrupt18.png)

我们知道Linux的c语言第一个入口为asmlinkage void __init start_kernel(void)，在这个函数中有调用函数：

![Alt text](/public/img/posts/interrupt19.png)

------

###3.4中断向量底层实现

异常，就是可以打断CPU正常运行流程的一些事情，比如外部中断、未定义指令、试图修改只读的数据、执行swi指令（Software Interrupt Instruction，软件中断指令）等。当这些事情发生时，CPU暂停当前的程序，先处理异常时间，然后再继续执行被中断的程序。操作系统中经常通过异常来完成一些特定的功能。其中的中断也占有很大的一部分。例如下面的几种情况：

当CPU执行未定义的机器指令时将触发“未定义指令异常”，操作系统可以利用这个特点使用一些自定义的机器指令，他们在异常处理函数中实现。
当用户程序试图修改的数据或执行的指令不在内存中时，也会触发一个“数据访问中止异常”或“指令预取中止异常”，在异常处理函数中将这些数据或指令读入内存，然后重新执行被中断的程序，这样可以节省内存，还使得操作系统可以运行这类程序，他们使用的内存远大于实际的物理内存。

在原先的内核版本中，内核在start_kernel函数（源码在init/main.c中）中调用trap_init、init_IRQ两个函数来设置异常和处理函数。在Linux最新的内核版本中，trap_init函数的内容发生了变化，在trap.c中：

```
void __init trap_init(void)
{
    return;
}
```

可见这个函数已经不起作用了，换成了early_trap_init();函数并且调用时间提前到了start_kernel函数的setup函数里面执行。

![Alt text](/public/img/posts/interrupt20.png)

![Alt text](/public/img/posts/interrupt21.png)

![Alt text](/public/img/posts/interrupt22.png)

![Alt text](/public/img/posts/interrupt23.png)

------

**中断跳转代码：**

![Alt text](/public/img/posts/interrupt24.png)

![Alt text](/public/img/posts/interrupt25.png)

![Alt text](/public/img/posts/interrupt26.png)

![Alt text](/public/img/posts/interrupt27.png)

![Alt text](/public/img/posts/interrupt28.png)

![Alt text](/public/img/posts/interrupt29.png)

![Alt text](/public/img/posts/interrupt30.png)

![Alt text](/public/img/posts/interrupt31.png)

![Alt text](/public/img/posts/interrupt32.png)

![Alt text](/public/img/posts/interrupt33.png)

![Alt text](/public/img/posts/interrupt34.png)

![Alt text](/public/img/posts/interrupt35.png)

![Alt text](/public/img/posts/interrupt36.png)

![Alt text](/public/img/posts/interrupt37.png)

![Alt text](/public/img/posts/interrupt38.png)

![Alt text](/public/img/posts/interrupt39.png)

![Alt text](/public/img/posts/interrupt40.png)
 