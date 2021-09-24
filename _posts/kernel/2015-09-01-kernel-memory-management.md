---
layout: post
title: "kernel learn-->Memory Management"
description: "内核 内存管理"
category: kernel
tags: [kernel]
---

##Memory Management

###Pages
The kernel treats physical pages as the basic unit of memory management.Although the
processor’s smallest addressable unit is a byte or a word, the memory management unit
(MMU, the hardware that manages memory and performs virtual to physical address
translations) typically deals in pages.Therefore, the MMU maintains the system’s page
tables with page-sized granularity (hence their name). In terms of virtual memory, pages
are the smallest unit that matters.

Most 32-bit architectures have 4KB pages, whereas most 64-bit architectures have 8KB pages.This implies that on a machine with 4KB pages and 1GB of memory, physical memory is divided into 262,144 distinct pages.

The kernel represents every physical page on the system with a struct page structure.

###Zones
Because of hardware limitations, the kernel cannot treat all pages as identical. Some pages,
because of their physical address in memory, cannot be used for certain tasks. Because of
this limitation, the kernel divides pages into different zones.The kernel uses the zones to
group pages of similar properties.

###Getting Pages

```
struct page * alloc_pages(gfp_t gfp_mask, unsigned int order)
```

This allocates 2order (that is, 1 << order) contiguous physical pages and returns a
pointer to the first page’s page structure; on error it returns NULL.

```
void * page_address(struct page *page)
```

This returns a pointer to the logical address where the given physical page currently
resides.

```
unsigned long __get_free_pages(gfp_t gfp_mask, unsigned int order)
```

This function works the same as alloc_pages(), except that it directly returns the
logical address of the first requested page. Because the pages are contiguous, the other
pages simply follow from the first.

###Getting Zeroed Pages

```
unsigned long get_zeroed_page(unsigned int gfp_mask)
```

This function works the same as __get_free_page(), except that the allocated page is
then zero-filled—every bit of every byte is unset.

This is useful for pages given to userspace
because the random garbage in an allocated page is not so random; it might contain
sensitive data.

###Freeing Pages

```
void __free_pages(struct page *page, unsigned int order)
void free_pages(unsigned long addr, unsigned int order)
void free_page(unsigned long addr)
```

```
unsigned long page;
page = __get_free_pages(GFP_KERNEL, 3);
if (!page) {
/* insufficient memory: you must handle this error! */
return –ENOMEM;
}
/* ‘page’ is now the address of the first of eight contiguous pages ... */
And here we free the eight pages, after we are done using them:
free_pages(page, 3);
/*
* our pages are now freed and we should no
* longer access the address stored in ‘page’
*/
```

It therefore often makes sense to allocate
your memory at the start of the routine to make handling the error easier.

These low-level page functions are useful when you need page-sized chunks of physically
contiguous pages, especially if you need exactly a single page or two. For more general
byte-sized allocations, the kernel provides kmalloc().

kmalloc()
The kmalloc() function’s operation is similar to that of user-space’s familiar malloc()
routine, with the exception of the additional flags parameter.The kmalloc() function is
a simple interface for obtaining kernel memory in byte-sized chunks.


逻辑地址

```
void * page_address(struct page *page)
```

###Slab Layer
Allocating and freeing data structures is one of the most common operations inside any kernel.
To facilitate frequent allocations and deallocations of data, programmers often introduce free lists.

If you frequently create many objects of the same type, consider using the slab cache.

kernel stacks are either one or two pages, depending on compile-time
configuration options.The stack can therefore range from 4KB to 16KB. Historically,
interrupt handlers shared the stack of the interrupted process.When single page stacks are
enabled, interrupt handlers are given their own stacks.

In any given function, you must keep stack usage to a minimum.There is no hard and fast
rule, but you should keep the sum of all local (that is, automatic) variables in a particular
function to a maximum of a couple hundred bytes.

Because the kernel does not make any effort to manage
the stack, when the stack overflows, the excess data simply spills into whatever exists at
the tail end of the stack.

In this chapter, we studied how the Linux kernel manages memory.We looked at the various
units and categorizations of memory, including bytes, pages, and zones. (Chapter 15
looks at a fourth categorization, the process address space.) We then discussed various
mechanisms for obtaining memory, including the page allocator and the slab allocator.
Obtaining memory inside the kernel is not always easy because you must be careful to
ensure that the allocation process respects certain kernel conditions, such as an inability to
block or access the filesystem.To that end, we discussed the gfp flags and the various use
cases and requirements for each flag.The relative difficulty in getting hold of memory in
the kernel is one of the largest differences between kernel and user-space development.
While much of this chapter discussed the family of interfaces used to obtain memory, you
should now also wield an understanding of why memory allocation in a kernel is difficult.