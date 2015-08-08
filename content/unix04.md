+++
date = "2015-08-07T17:00:00+02:00"
title = "Locks and concurrency"
+++

This is the fifth post in my series on Grokking xv6. We look at
concurrent processes, what locks are and how they can used, and
finally we look at how locks are implemented in xv6.

<!--more-->

In the most straightforward mental model of how code executes, you
probably think of lines of code as executing one after another,
serially. This is correct for single-processor CPUs, but as soon as
you have multiple cores this assumption is no longer true.

Let's say we have two *threads of execution*, A and B, executing two
statements each.

```
Thread A    Thread B
a1          b1
a2          b2
```

The only guarantees we can make are that a1 happens before a2, `a1 <
a2`, and `b1 < b2`. We have many possibilties for the event ordering
between all of the events:

```
a1 < a2 < b1 < b2
a1 < b1 < a2 < b2
a1 < b1 < b2 < a2
b1 < a1 < a2 < b2
b1 < a1 < b2 < b2
b1 < b2 < a1 < a2
```

All of these possible event orderings can - and probably will -
happen, given that the program is running for a long enough time
period. This *non-determinsm* makes it hard to reason about and debug
*concurrent* code.

There are many ways to be clever and rigorous about concurrent code
using various *concurrency primitives*, but we are just going to talk
about one: the *lock*, or *mutex* (from *mutual exclusion*). What do
all these concurrency primitives do? They provide concurrent control,
just like there are if-statements and while loops in non-concurrent
code.

## Locks

Normally it's a good idea to write code that doesn't use *shared
memory*. In the example above, if none of A's code depends on B's code
and vice versa, it doesn't matter what the exact ordering is. This is
the common case when we have multiple processes running.

In user-space, the only reason to run a program on multiple cores is
performance, and specifically when your program is CPU-bound as
opposed to being disk or I/O-bound.

In the kernel there is *shared memory* everywhere - process tables,
file tables, memory free lists, etc. Even if we have only a single
core, *hardware interrupts* (for example, when you press a key) change
the flow of execution in non-obvious ways. By using locks we can
protect a piece of code by maintaining important *invariants* over it.

How do locks work? Let's say we have a *chained hash table* with N
buckets that each have a linked lists of entries, and we want to run
with multiple threads for performance reasons. We might notice that
there's a problem with missing entries when we let multiple threads
modify the hash table, so we can put a lock around this *critical
section*:

```
    // code from ph.c in my xv6 repo

    pthread_mutex_lock(&lock);
    put(keys[b*n + i], n);
    pthread_mutex_unlock(&lock);
```

Here the critical section is the line between lock and unlock
statements. Because the lock is locked only a single thread can enter
into the critical section at any given time. This way we don't get any
*lost updates* but we still get performance gains, because we can get
entries in parallel.

**Problem for the author**: In a lecture Robert Morris seems to say
  there are problems with not locking the getting part too. I don't
  see how. Did I misunderstand what he was saying or is there an
  insight I'm missing? See note at the end for an answer.

This is what is called a *coarse-grained* lock. We can do
*fine-grained* locks to increase performance, at the cost of making
the implementation a bit trickier. For example, in the hash table
example, we can put a lock on each bucket in the put function:

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
temp  = count
count = temp + 1
```

We can't make any guarantees that these lines will happen
sequentially, and if another process is using the same variable you
are in for trouble. This highlights the importance of *atomicity*, and
in this case we could've solved the problem with a simple lock.

## How are locks implemented in xv6?

There are many ways to implement locks, and xv6 implements them as
*spinlocks*. This means the CPU is spinning in a loop, waiting for the
lock to be released. This is great for short-term locks, but horrible
for long-term locks, which is why other implementations of locks are
frequently used. Here's an excerpt from the relevant code in xv6:

```
// Mutual exclusion lock.
struct spinlock {
  uint locked;       // Is the lock held?
};
```

As you can see it's really just an (unsigned) integer that signifies
if it's held or not. If it is zero the lock is available. The magic is
in its two primary functions - `acquire` and `release` (or lock and
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

There are two main things to note here. The first is the `pushcli`
call. This disables hardware interrupts, which we do to ensure that we
execute the code in `acquire` sequentially. If we didn't, a hardware
interrupt might change the flow of execution so that another piece of
code tries to acquire the lock, and things would quickly get messy.

The `release` function essentially does the opposite of `acquire`, and
it ends by enabling hardware interrupts again. Because locks can be
nested - we can hold a lock around a function which has another lock
inside of it - it's not enough to just enable and disable hardware
interrupts directly. We maintain the ordering of locking and unlocking
by using a stack.

The second thing to note is the `xcgh` instruction, which is a
*compare-and-swap* instruction. Two CPUs could both notice that the
lock is free, and both try to grab it. To avoid this, the check and
changing of the locked field has to be atomic, i.e. happen in one
single step. `xchg` is a special instruction that provides this
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

(If you liked this, you might enjoy
[Grokking xv6](http://experiments.oskarth.com/unix00/). To stay up to
date on my experiments, consider [subscribing](http://eepurl.com/bvtdfj).)

*NOTE: Kamal Marhubi pointed out that rtm seems to be saying three
  things: (1) In this program, all gets happen after all puts so it's
  definitely not an issue. (2) There's a problem with get operations
  in the general case, because we could get an entry in a
  non-consistent state (3) In this specific implementation it's not a
  problem, because the pointer to the newly-created entry is changed
  atomically, so the worst we can do is get a stale item. Regardless,
  these things are tricky and it's probably better to lock the get
  operations as well to avoid subtle bugs.*
