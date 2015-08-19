+++
date = "2015-08-19T16:00:00+02:00"
title = "Grok LOC?"
+++

This is the seventh post in my series on Grokking xv6. In this post we
will look at how I tested one of my original hypothesis starting out:
understanding every single line of code.

<!--more-->

The last two months I have been going through xv6. Here's a quote on
one of my hypotheses from my
[initial post](http://experiments.oskarth.com/unix00/):

> 1. With dedicated study, I’ll be able to grok xv6 in about a month.
> 
> To make the concept of grokking more concrete:
>
> * understand every single line in the source code
> * teach it or parts of it to someone else
> * rewrite or extend parts of the source code

In this post I want to talk specifically about understanding every
single line of code. Before we go into that there's two observations I
want to make:

1. **Time frame**: I originally set out to go through xv6 in about a
   month. In reality it took close to two months, and not counting
   vacation around six weeks. This is still "about" a month but was
   a bit longer than expected and desired.

2. **Every single line of code**: I realized after a while that this
   was a bit over-ambitious so I mentally adjusted the bar to
   understanding 95% of the code.

## Methodology

With that said, how do we test a hypothesis like this? And what does
it mean to understand a line of code? I decided to use a sampling
method and classify lines of code into ones I either understood or
didn't. This test was somewhat subjective - essentially I asked myself
if I understood what a line's purpose was, and if I understood all the
components of the line. For example, it was fine to check how related
functions were defined and to look up basic documentation, but
anything that required googling was a 'No'. The idea is that if I
wrote a piece of code myself from scratch this is about what I would
expect to understand.

Since the xv6 code booklet was numbered with lines of code from about
1 to 10000, I simply generated a bunch of random numbers from 1 to
10000:

```
(repeatedly 50 #(rand-int 10000)) ; Clojurescript
```

This ensured *uniform sampling*. Next I removed all the lines that
matched one of the following: empty lines, comments, `#include`
statements, and lines of code with very few characters in them, such
as `{` or `}`. The goal of this was to get real lines of code only,
and not ones that have a very generic explanation.

I repeated the above process until I had 20 lines of real code. 20 was
chosen as being a good balance between getting enough of a sample for
my purposes, and not taking too long to go through.

Since I would only be checking a *sample* as opposed to the whole
*population* (i.e. all ~ 10 000 lines of code), I would only get an
estimate for my understanding of the code.

I decided that I could consider my hypothesis false if the observed
value was more than two *standard deviations* away (outside a 95%
*confidence interval*) from the *expected value*. Assuming the
hypothesis that I would understand 95% of the code and that I would
look at 20 lines of code, I used a little utility I wrote,
[rrange](https://github.com/oskarth/rrange), to see what the range
would be.

```
~$ rrange 0.95 20    # unix util
Around 19 ~ [17, 20]
```

This means that if I have a 95% understanding of the whole code base
and I look at 20 random lines of code, I should expect to understand
17 to 20 of them.

When making statistical claims such as this, one has to be careful
about not confusing the sample and population. For example, if I have
a real understanding of 70% of the code, it wouldn't be that unlikely
to get 18 out of 20 'Yes's'.

```
~$ rrange 0.7 20
Around 14 ~ [10, 18]
```

Another potential source of error is that I performed the test on
myself, and I have a vested interest in getting a good outcome. A more
objective test would be desirable.

## Results

Here's a list of the 20 samples, together with a note on its context
and a brief comment on my understanding of the line and its
purpose. Sometimes, in the case of a return statement, the test is
whether I understand why that thing is being returned.

<pre>
Result: 18/20

1
5845 return fd;
CONTEXT: sysfile.c: fdalloc function
COMMENT: Return file descriptor, maps to file for a process.
VERDICT: YES

2
8943 orl $CR0_PE, %eax
CONTEXT: bootasm.S: seta20.2 function, real to protected mode
COMMENT: Bitwise logical OR load of protection mode into temp reg.
         Part of enabling protected mode cr0 register.
VERDICT: YES (after a few mins of looking up documentation)

3
5772 n1 = max;
CONTEXT: file.c: filewrite function
COMMENT: n1 is number of bytes left to write, capped by max.
VERDICT: YES

4
7168 lapicw(LINT0, MASKED);
CONTEXT: lapic.c: lapicinit
COMMENT: Part of dealing with interrupts, but don't know details.
VERDICT: NO

5
8785 return cmd;
CONTEXT: sh.c: parseredirs function
COMMENT: After parsing a redir command we want to use it.
VERDICT: YES

6
2819 release(lk);
CONTEXT: proc.c: sleep function
COMMENT: When a process sleeps we don't want to block other processes.
VERDICT: YES

7
3707 return −1;
CONTEXT: sysproc.c: sys_sbrk function
COMMENT: Address of function argument out of bound, see fetchint.
VERDICT: YES

8
5970 panic("isdirempty: readi");
CONTEXT: sysfile.c: isdirempty function
COMMENT: Panics if reading data for an dir entry isn't right size.
VERDICT: YES

9
5906 if(argfd(0, 0, &f) < 0 || argptr(1, (void*)&st, sizeof(*st)) < 0)
CONTEXT: sysfile.c: sys_fstat function
COMMENT: Something wrong with one of the two arguments to fstat.
VERDICT: YES

10
5659 return f;
CONTEXT: file.c: filedup function
COMMENT: Return file after increasing its reference count.
         Used when forking process for copying open files table.
VERDICT: YES

11
3536 ep = (char*)proc−>sz;
CONTEXT: syscall.c: fetchstr function
COMMENT: Don't look for end of string above proc's address space.
VERDICT: YES (after a few minutes of looking up documentation)

12
3520 return −1;
CONTEXT: syscall.c: fetchint function
COMMENT: Error, can't read argument above proc's memory.
VERDICT: YES

13
7964 return −1;
CONTEXT: console.c: consoleread function
COMMENT: If console process is killed, return error code.
VERDICT: YES

14
8428 break;
CONTEXT: sh.c: runcmd function
COMMENT: Something went wrong, exit if exec function returns.
VERDICT: YES

15
6236 if(i >= NELEM(argv))
CONTEXT: sysfile.c: sys_exec function
COMMENT: Args read from stack, if they exceed capacity it's an error.
VERDICT: YES

16
6331 if(elf.magic != ELF_MAGIC)
CONTEXT: exec.c: exec function
COMMENT: Error if ELF file doesn't have magic constant in right place.
VERDICT: YES

17
2906 static char *states[] = {
CONTEXT: proc.c: procdump function
COMMENT: Array of designated initializers with names for proc's state.
VERDICT: YES

18
0771 ((uint)(base) >> 16) & 0xff, type, 1, dpl, 1, \
CONTEXT: mmu.h: SEG macro
COMMENT: Part of casting some x86 segment to a struct. Bit magic.
VERDICT: NO

19
5241 st−>type = ip−>type;
CONTEXT: fs.c: stati function
COMMENT: Copy inode's type to stat's type for filestat system call.
VERDICT: YES

20
5776 if ((r = writei(f−>ip, addr + i, f−>off, n1)) > 0)
CONTEXT: file.c: filewrite function
COMMENT: Write n1 bytes from addr + i to inode, stored at I/O offset.
         Returns bytes written, data stored in data blocks.
VERDICT: YES
</pre>

You can find the source code
[here](http://pdos.csail.mit.edu/6.828/2014/xv6/xv6-rev8.pdf).

## Conclusion and further work

My understanding of the sample code was within two standard deviations
of the estimated value, so I failed to reject the hypothesis that I
understand 95% of the xv6 source code. Furthermore, it seems unlikely
that I understand less than 70% of the source code.

Initially I was skeptical about my ability to test this hypothesis,
but I'm pretty happy with the method used in this article. A few weeks
ago I did a trial run, and I found that I got more 'No's' on samples
related to the filesystem, which I had yet to study by then. A similar
number of samples related to the file system were present this time
around, and I got more 'Yes's' on those samples, which seems to
reflect my deepened understanding of that part of the code base. This
suggests that the test for understanding that I'm using is not
completely unreasonable.

The test still leaves a lot to desire though, primarily because of two
reasons: (a) it lacks objectivity, and (b) it doesn't touch on the
essence of programming. The essence of programming is to program, as
opposed to reading other people's programs. A different direction that
I think would be interesting to pursue is to re-create an OS or part
of it from scratch. However, I think this test, along with the related
homework assignments and the other posts in this series, are good
enough for my present purposes.

We are almost coming to an end to the series. The next step will be to
use my knowledge and do something new with it.

(If you liked this, you might enjoy
[Writing a Lisp Interpreter](http://experiments.oskarth.com/lisp-interpreter/). To
stay up to date on my experiments, consider
[subscribing](http://eepurl.com/bvtdfj).)
