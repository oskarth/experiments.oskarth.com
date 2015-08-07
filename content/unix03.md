+++
date = "2015-07-18T18:00:00+02:00"
title = "Page tables and virtual memory"
+++

This is the fourth post in my series on Grokking xv6. We look at page
tables, virtual memory and how lazy page allocation works.

<!--more-->

When we see *memory addresses* being printed in *GDB*, these aren't
the *physical addresses* that the *hardware* deals with. Instead they
are *virtual addresses*. Virtual addresses are useful in providing
*process isolation*, which is one of the key goals of *operating
systems*. Here's a very rough sketch of what memory looks like for two
processes:

```
+--------+                 +--------+
| Kernel |                 | Kernel |
|--------| <- 0x80000000   |--------| <- 0x80000000
| User 1 |                 | User 2 |
+--------+ <- 0x00000000   +--------+ <- 0x00000000
```

For process 1, its *user space* starts at `0x00000000`. It can't
access process 2's memory. For process 2 it's the same. The actual
physical addresses for each process's memory are different, but their
virtual addresses look the same.

The *kernel space* is the same for both processes and starts at
`0x80000000` (in xv6 at least, but the principle is the same). Seen
from the point of view of one process, its user space and kernel space
makes up its *address space*.

The *mapping* between virtual addresses and physical addresses is done
by *page tables*. When the operating system switches to *executing*
another process, it changes to that process's page table. Abstractly,
a page table is a mapping from virtual addresses to physical
addresses.

Kernel space is part of a process's address space because a process
might do a lot of system calls, switching from executing in kernel
space and user space, and we want to avoid doing page table switches
every time we make a system call.

The smallest piece of memory we can allocate is called a *page*. A
page in xv6 and x86 is normally a chunk of 4096 (2^12) bytes of
contiguous memory. In this case, 4096 is the *page size*.

Conceptually speaking, a page table can be thought of as an array of
2^20 (roughly a million) items. We could imagine storing pages
directly in this array, but for one process alone this would take up
4GB of memory, which would be quite bad.

## Page table as a tree

We can solve this problem by creating a tree-like structure to reduce
the amount of physical memory being used. The page table as an array
can be conceptualized as a zero-level tree, where all the information
is at the top level. We could also imagine a one-level tree where the
array simply contains *pointers* to pages. In xv6 there's a two-level
structure that works as follows. A virtual address, such as
`0x0df2b000` corresponds in binary to
`1101111100101011000000000000`. This 32-bit address can be decomposed
into the following three parts:

```
    10      10       12
  | Dir | Table | Offset |
```

The abstract concept of a page table can be implemented as follows. In
the virtual address above, the first 10 bits are used as an index into
a so called *page directory*. If a corresponding *page directory
entry* (PDE) exists, it uses the next 10 bits to look up a *page table
entry* (PTE). A PTE consists of a 20-bit *physical page number* (PPN)
and some flag bits that gives some information about how this piece of
memory can be used. The PPN refers to the actual physical location of
the memory. If either the page directory entry or the PTE doesn't
exist we get a *page fault* which tells us that the memory location we
are trying to access hasn't been mapped to physical memory.

If the page directory entry and PTE exist, the *paging hardware*, as
it looks up what is in that actual memory location, replaces the top
20 bits with the PPN to get the actual physical address. The 12 last
bits are left unchanged, and they correspond to the offset within the
page.

It might be a bit difficult to wrap your head around this tree-like
structure at first. One way to think of it as looking up cities in a
list of cities around the world, and then looking up a phone number in
that city's phone book. If a city doesn't exist, there's obviously no
reason to have an empty phone book for it.

## Lazy page allocation

There's some neat tricks you can do with the page tables. One of them
is lazy page allocation. The idea is that you don't allocate memory
before you need it, but instead you try to access it and then, when
you get a page fault, you allocate memory at the very last
moment. This can be useful if you have a very large *sparse* array,
because it will only take up as much space as is needed.

In code lazy page allocation might look something like this, without
error handling:


```
if(tf->trapno == T_SYSCALL) {
   // code to deal with a system call in kernel space
}
if(tf->trapno == T_PGFLT) {
  uint a = PGROUNDDOWN(rcr2()); // round down faulty VA to page start
  char *mem;
  mem = kalloc();
  memset(mem, 0, PGSIZE);
  mappages(proc->pgdir, (char*)a, PGSIZE, v2p(mem), PTE_W|PTE_U);
  return;
  }
```

When we get notified of a page fault, we take the faulting address and
round it down to the closest page boundary (i.e. evenly divided by or
page size). We then ask for a page of physical memory and zero that
memory. Finally we map the virtual address to that piece of physical
memory in the process's page table, using flags that let us know this
memory belongs to a user (as opposed to the kernel) and that the
memory is writable.

In more sophisticated operating systems like Linux you might see
things like 4-level page tables. The general principle is the same as
we discussed though - it's a clever way of mapping virtual addresses
to physical addresses. The main differences is in terms of *caching*
frequently accessed pages or page directories.
