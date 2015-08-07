+++
date = "2015-08-01T10:56:57+02:00"
draft = true
title = "Locks and synchronization"

+++

This is the fifth post in my series on Grokking xv6. We look at
concurrent processes, what locks are and how they can used, and
finally we look at how locks are implemented in xv6.

<!--more-->

In computer science, *synchronization* is a relationship in time
 between many events. The most common ones are *serialization*, events
 happening one after another, and *mutual exclusion*, events aren't
 allowed to happen at the same time.

## Non-determinism

Let's make it a bit more concrete. In your mental model of how code
executes, you probably think of lines of code as executing one after
another, serially. This is correct for single-processor CPUs, but as
soon as you have multiple cores this assumption is no longer true. In
fact, the best way to think of it is to assume the worst ordering of
events as possible.

```
Thread A    Thread B
a1          b1
a2          b2
```

In the above example, let's say we have two threads executing a series
of statements. The only guarantees we can make is that a1 happens
before a2, `a1 < a2`, and `b1 < b2`. We have many possibilties for the
event ordering between all of the events:

```
a1 < a2 < b1 < b2
a1 < b1 < a2 < b2
a1 < b1 < b2 < a2
b1 < a1 < a2 < b2
b1 < a1 < b2 < b2
b1 < b2 < a1 < a2
```

All of these possible events can - and will - happen. This is a form
of *non-determinsm* that makes it hard to reason about and debug
*concurrent* code. There are many ways to be clever and rigorous about
concurrent code using various concurrency primitives, but we are just
going to talk about one specific primitive: the lock, or mutex (from
mutual exclusion). What do all these concurrency primitives do? They
provide concurrent control, just like there are if-statements and
while loops in "normal" code. If you want to read more about this, I
suggest you check out *The Little Book Of Semaphores* by Downey, which
is free online.

## Locks

There's something we didn't mention, which is why bother with at all?
There are two answer to that. First, in user-space, processes can run
in independently *threads of execution* and if another process jumps
in the ordering doesn't matter, since they have a separate address
space and everything. So the only reason to run a program on multiple
cores is performance, and specifically that your program is CPU-bound
as opposed to being disk or I/O-bound. Only then it might be worth the
effort.

The other reason is in the kernel, where there are *shared variables*
everywhere (process tables, file tables, memory free lists, etc) and
we have to watch out for *hardware interrupts*. These interrupts, as
hinted at in previous posts, happen even if we have just one core
running, so we need to care there as well.

How do locks work? Let's say we have a *chained hash table* with N
buckets that each have a linked lists of entries, and we want to run
with multiple threads for performance reasons. We might notice that
there's a problem with missing entries when we let multiple threads
modify the hash table, so we can put a lock around this *critical
section*:

```
    (code from ph.c in my xv6 repo)

    pthread_mutex_lock(&lock);
    put(keys[b*n + i], n);
    pthread_mutex_unlock(&lock);
```

Here the critical section is the line between lock and unlock
statements. Because the lock is on only a single thread can enter into
the critical section at any given time. This way we don't get any
*lost updates* but we still get performance gains, because we can get
entries in parallel. I did some basic profiling to see the speed-up
you get for various options.

TODO: Maybe remove this part, too much to explain about this specific
implementation.

```
label      nthreads work time  missing-keys
---------------------------------------------------
incorrect  1        x1   ~6s   0
incorrect  2        x2   ~7s   ~17k
incorrect  4        x4   ~7s   ~40k
lock both  1        x1   ~6s   0
lock both  2        x2   ~19s  0
lock both  4        x4   ~38s  0
lock put   1        x1   ~6s   0
lock put   2        x2   ~7s   0
lock put   4        x4   ~7s   0
```

**Problem for the author**: in a lecture Robert Morris seems to say
  there are problems with not locking the getting part too. I don't
  see how. Did I misunderstand what he was saying or what is the
  missing insight?

This is what is called a *coarse-grained* lock. We can do
*fine-grained* locks to increase perfomance, at the cost of making the
implementation a bit trickier. For example, in the hash table example,
we can put a lock on each bucket in the put function:

```
  int i = key % NBUCKET;
  pthread_mutex_lock(&lock[i]);
  insert(key, value, &table[i], table[i]);
  pthread_mutex_unlock(&lock[i]);
  ```

This way multiple cores can update the hash table simultaneously,
without disrupting each other.

A final word of warning. Reasoning about concurrent code can be
tricky, and you can't even assume a single line will happen
sequentially. For example `count++` is actually two (assembly)
instructions in one. In C we might say it corresponds to:

```
temp = count
count = temp + 1
```

And we can't make any guarantees that these lines will happen
sequentially, and if another process is using the same variable you
are in for trouble. This highlights the importance of *atomicity*, and
in this case we could've solved the problem with a simple lock.

## How are locks implemented in xv6?

There are many ways to implement locks, and xv6 implements them as
*spinlocks*. This means the CPU is spinning, waiting for the lock to
be released. This is great for short-term locks, but horrible for
long-term locks, which is why other locks are frequently used. Here's
an excerpt from the relevant code in xv6:

```
// Mutual exclusion lock.
struct spinlock {
  uint locked;       // Is the lock held?
};
```

As you can see it's really just an (unsigned) integer that signifies
if it's held or not. If it is zero the lock is available. The magic is
in its two primary functions - acquire and release (or lock and
unlock):

```
void acquire(struct spinlock *lk) {
  pushcli(); // disable interrupts to avoid deadlock.
  if(holding(lk)) // checks if the lock is locked.
  panic("acquire"); // we can't acquire an already acquired lock.
  
  while(xchg(&lk->locked, 1) != 0) // xchg is atomic.
    ;
}
```

There are two main things to note here. The first is the pushcli
call. They disable and enable hardware interrupts,
respectively. Because locks can be nested (i.e. we can hold a lock
around a function, and inside that function there's another lock),
it's not enough to just enable and disable them directly, but instead
it's put on a stack that remembers the nesting of locking and
unlocking. Why do we disable interrupts? Because otherwise we might
end up in some trap handler somewhere when the user press a key, or 10
ms has passed, etc. This would leave our lock code in an inconsistent
state, which we don't want.

The second thing is the xcgh instruction. Two CPUs could both notice
that the lock is free, and both try to grab it. To avoid that, the
check and changing of the locked field has to be atomic, i.e. happen
in one single step. xchg is a special instruction that provides this
guarantee. The while-loop is spinning, checking if it can acquire the
lock and constantly "updating" the locked field to 1. Once the locked
field is set to 0, i.e. the lock has been released somewhere else,
xchg checks this field and updates the field to 1, all in one step.

There's a lot more to concurrency and locks than what we have covered
in this post, but hopefully this has given you a taste of what the
landscape looks like and what some of the challanges are. If you want
to learn more about concurrency primitives from a conceptual and
programming point of view, I highly recommend *The Little Book of
Semaphores* by Downey, which is available for free online.
