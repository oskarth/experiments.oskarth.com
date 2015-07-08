+++
date = "2015-07-04T13:42:52+02:00"
draft = true
title = "What's on the stack?"

+++

This is the third post in my series on Grokking xv6. We will look at system
calls and how they work under the hood.

<!--more-->

## Introduction

TODO: Primer on reading hexadecimal

A word of warning: in this article there'll be a lot of things that
you'll see that we won't explain. Instead, we are going to learn how
to squint at the code, suppress detail and focus on what we want to do.

Hardest part for me was not having a good mental model of the stack
and how it operates, attempt to give this to you.

Three purposes:

1. See how a system call flows through the system.
2. Learn how to use GDB to figure such things out.
3. Learn about the stack.

A computer executes instructions, one after another.


## Running ls

Even if you don't understand all you can understand the gist of
it. Let's see what we might recognize.


This is the main function of ls, in ls.c:

```
int
main(int argc, char *argv[])
{
  int i;

  if(argc < 2){
    ls(".");
    exit();
  }
  for(i=1; i<argc; i++)
    ls(argv[i]);
  exit();
}
```

Here's the equivalent asssembly for that main file.

(Using ATT syntax, not Intel, see Brennan).

General format: mnemonic source, destination. This is (usually)
straight to binary.

Assembly language is very simple; every single instruction is easily
understandable. The problem is understanding the context and the
vastness of it. If you just go line by line and take your time to look
up in Carter and Brennan, you will know exactly what is going on.

```
(gdb) disassemble main
Dump of assembler code for function main:
=> 0x00000304 <+0>:     push   %ebp
   0x00000305 <+1>:     mov    %esp,%ebp
   0x00000307 <+3>:     and    $0xfffffff0,%esp
   0x0000030a <+6>:     sub    $0x20,%esp
   0x0000030d <+9>:     cmpl   $0x1,0x8(%ebp)
   0x00000311 <+13>:    jg     0x324 <main+32>
   0x00000313 <+15>:    movl   $0xb5f,(%esp)
   0x0000031a <+22>:    call   0xb0 <ls>
   0x0000031f <+27>:    call   0x5cb <exit>
   0x00000324 <+32>:    movl   $0x1,0x1c(%esp)
   0x0000032c <+40>:    jmp    0x34d <main+73>
   0x0000032e <+42>:    mov    0x1c(%esp),%eax
   0x00000332 <+46>:    lea    0x0(,%eax,4),%edx
   0x00000339 <+53>:    mov    0xc(%ebp),%eax
   0x0000033c <+56>:    add    %edx,%eax
   0x0000033e <+58>:    mov    (%eax),%eax
   0x00000340 <+60>:    mov    %eax,(%esp)
   0x00000343 <+63>:    call   0xb0 <ls>
   0x00000348 <+68>:    addl   $0x1,0x1c(%esp)
   0x0000034d <+73>:    mov    0x1c(%esp),%eax
   0x00000351 <+77>:    cmp    0x8(%ebp),%eax
   0x00000354 <+80>:    jl     0x32e <main+42>
   0x00000356 <+82>:    call   0x5cb <exit>
```

- How to read this? Format explain.

Look at the assembly, what do we see? We can see that there's calls to
<ls> and <exit>, and then later a call to <ls>, and then at the very
end another call to <exit>. These are the function call we can see in
the C code, so we have already learned something about how to read assembly.

### The Registers EIP, ESP and EBP and the Stack

Registers store data for processor. There are many types of these, but
we are going to focus on three which are essential in understanding
how things move around. Note: there are many diff names depending on
arch, IP->EIP->RIP.

```
(gdb) info reg eip esp ebp
eip            0x304    0x304 <main>
esp            0x2fe8   0x2fe8
ebp            0x3fb8   0x3fb8
```

This tells us what is inside these three registers. 0x means it's in
hexadecimal, so this just an integer, but it's essentially an index
for the computer. Sort of like saying "the forks are in the second
drawer".

EIP is the instruction pointer. This is where "the computer"
(processor) is executing right now.

The numbers are hexadecimal (base 16) addresses.

-x for inspect memory. explain it.

Note that eip is the same as the very first instruction (extra zeroes
are sometimes removed when printing). If we run `x /4i $eip` we see
the coming 4 instructions. Since we are at the very beginning of main,
these are the same as the assembly code generated above by
`disassemble`.

```
(gdb) x /4i $eip
=> 0x304 <main>:        push   %ebp
   0x305 <main+1>:      mov    %esp,%ebp
   0x307 <main+3>:      and    $0xfffffff0,%esp
   0x30a <main+6>:      sub    $0x20,%esp
```

A stack is LIFO - last in first out. Two main operationd - push and pop.

(We also have CALL and RET.)

(These are calling conventions, not universal ~ when no apply? gist tho.)

Let's look at the two first lines of that assembly.

```
push   %ebp
mov    %esp,%ebp
```

push adds data to the stack. What does this mean?

TODO: explain basics of stack here or why we might want it?  when we write
programs, we don't (anymore) write it just as a series of instructions
of the computer to execute. Instead, we use "structured programming"
or subfunctions that call each other. If we follow the execution of a
code by pointing at the screen we might say "this calls this, which
calls this, then it comes back here". The stack is how the computer
keeps track of things like, where it came from, what its arguments
are, etc.

We can step one instruction at a time by `si`. If we do this, we just
performed the push (explain that the thing we see is what we are about
to do) and we should expect the stack to have changed.

(can use $eip-1 to show context)

The stack pointer points to the stack, and indeed we see:

```
(gdb) x /x $esp
0x2fe4: 0x00003fb8
```

0x2f34 is the location of the stack pointer and it contains the
address of the base pointer.

Wait, why did esp change? It was something else before this
instruction, and there was nothing in that line about $esp. That's
because PUSH instruction is a "macro" - in fact a push instruction can
be seen as two instructions in one.

```push $ebp``` is equivalent to

```
sub $4 $esp
mov $ebp ($esp)
```

(Register names have %.) This subtracts 4 from $esp, since 32-bit
machine and byte 8, 4*8=32. Just two numbers - pointers address bytes,
so you subtract 4 bytes from esp (x86-32, in 64 it'd be 8). The second
instruction moves the register $ebp into what the esp register points
to.

Like pointers in C, parantheses is a way to dereference what something
is pointing to. so $esp is the address of the stack pointer, and
($esp) is what the stack pointer points to.

To illustrate:
- (maybe have both before and after?)

The stack grows "down".

```
(gdb) x /x $esp
0x2fe8: 0xffffffff
```

After we have pushed $ebp:

```
(gdb) x /2x $esp
0x2fe4: 0x00003fb8      0xffffffff
```
(how to read that, pr do in two steps)

Illustration, mby old:

```
address | content                     (before)
-----------------
0x2fe8  | 0xfffffff  <-- ESP

address | content                     (after)
-----------------
0x2fe4: 0x00003fb8   <-- ESP

or with x/2x $esp:

address | content                     (after)
-----------------
0x2fe8  | 0xffffffff
0x2f34  | 0x00003fb8 <-- ESP
```

(Because main is the entry point of the program, there's nothing to
return to. 0xffffffff is a "fake PC" and is equivalent to -1 in two's
complement. This isn't essential for understanding.)

Ok, now $ebp is on "the stack", whatever that means. what about the second line?

```
mov    %esp,%ebp
```

This moves the esp register into ebp. Indeed:

```
(gdb) x /1x $ebp
0x2fe4: 0x00003fb8
```

There's something I didn't tell you. The arguments to main are also on
the stack!

```
(gdb) x /4x $esp
0x2fe4: 0x00003fb8      0xffffffff      0x00000001      0x00002ff4
```

We could also see this by printing out one at a time, decrementing
$esp - which is just an integer, by 4 using `x $esp+4`.

0x2ff4 is the first that was pushed, that's the argv, then argc, then
return address is 0xfff... which is -1, since nothing to return
to. Will see more when going to ls. Then we have ebp, which we just
pushed.

Can see if we call `info frame` too (haven't explained frames yet!).

```
(gdb) info frame
Stack level 0, frame at 0x2fec:
 eip = 0x305 in main (ls.c:75); saved eip = 0xffffffff
 source language c.
 Arglist at 0x2fe4, args: argc=1, argv=0x2ff4
 Locals at 0x2fe4, Previous frame's sp is 0x2fec
 Saved registers:
 ebp at 0x2fe4, eip at 0x2fe8
```

- (So why did we do that?)

We can run `step` to step through source code, so instead of
instruction by instruction we go line by line in C.

Talk about aligning the stack and making room for args? Let's see if
it happens again in ls, think so.

Keep that stringent: understanding the stack.

Next call / instruction:

`AND 0xfffffff0 $ESP` - this is called stack alignment. Using two's
complement (?), this is -16. ANDing that together with esp means $esp
will be at current or at 16 byte boundary. This is for "basic
optimization" (?) - or rather, for things like SIMD, but that's
outside of scope of article (https://en.wikipedia.org/wiki/SIMD /
https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions mby). You can
see that it's evenly divided by 16 with: `p 0x2fe0 % 16`.

(but why before? I don't really get it).

now stack looks like:
```
(gdb) x /6x $esp
0x2fe0: 0x00000000      0x00003fb8      0xffffffff      0x00000001
0x2ff0: 0x00002ff4      0x00002ffc
```

TODO: Also, ddin't talk about EBP! What is the purpose of EBP?

What about other thing? why sub 20 exactly? Make room for stuff, local variables.

```sub    $0x20,%esp```

Why 20?

big picture: it's a frame. :$

right before:

```
(gdb) x /6x $esp
0x2fe0: 0x00000000      0x00003fb8      0xffffffff      0x00000001
0x2ff0: 0x00002ff4      0x00002ffc
```

let's step!

yay
(gdb) p (char*)0xb5f
$7 = 0xb5f "."

Ok with `ls foo`


```
(gdb) x /5x $esp
0x2fe0: 0xffffffff      0x00000002      0x00002fec      0x00002ffc
0x2ff0: 0x00002ff8
```

```
(gdb) p (char*)0x00002ffc
$30 = 0x2ffc "ls"
(gdb) p (char*)0x00002ff8
$31 = 0x2ff8 "foo"
```
pretty cool! in reverse order: argv 2, 

- show print line by line

## A restart (integrate with above)

Want to break at main.

See this: ```Breakpoint 1, main (argc=2, argv=0x2fec) at ls.c:75```

What's on the stack?
(gdb) x /5x $esp
0x2fe0: 0xffffffff      0x00000002      0x00002fec      0x00002ffc
0x2ff0: 0x00002ff8

(gdb) x /8x $esp
0x2fe0: 0xffffffff      0x00000002      0x00002fec      0x00002ffc
0x2ff0: 0x00002ff8      0x00000000      0x006f6f66      0x0000736c

No idea what those other things are.

If we didn't have ls foo we'd just need top 4, like above.

- EDIT THIS SHIET.











Anyway, keep moving. Then delete.

- CDECL calling convention

- argc/arv?

- aligning the stack, and call



















### Let's add a syscall!


### Further resources

If you want to learn more, check out Carter and Brennan, and Cox.













1. Let's trace an ordinary system call.

Which? sleep? wait? getpid? mkdir? Looks reasonably simple. But maybe take
something like fstat that uses argptr.

sys_fstat, calls filestat, which just gets some metadata about a file. good
example methinks.

file.h has file struct, stat.h has a stat.

ALT look at procs. Files and procs are the most important parts, no?

What is the hardest part about understanding systems programming things? We will
look into the file system more later but.

What about the stack and these things? That's something you struggled with a
lot. Understanding what goes on GDB level.

Also, maybe I could make a live video of this? Just stepping through it.

So what would I want to see?

Stepping through GDB to understand what's at the stack. How does user and kernel
stack interplay? This is a big one, I think tonight after pairing.

WHATS ON THE STACK? GDB AND KERNELY STUFF.

Show, don't tell.

Multiple GDB sessions.


when we do ls, how do we get the actual info? where does it com from? inside
fstat etc.

suppress detail, but only the rightkind. zoom in on fstat.

similar fstat. Later tonight.
explian things here
root@xv6:~/xv6# grep fstat *.[chS]

GDB stuff:
https://www.youtube.com/watch?v=WJaTW8RWrLw
https://www.eecs.umich.edu/courses/eecs373/readings/Debugger.pdf
http://heather.cs.ucdavis.edu/~matloff/UnixAndC/CLanguage/Debug.html

http://beej.us/guide/bggdb/


What was missing from the last one? Divide it up into pieces?

When is a system call invoked?


function syscall that takes checks if it is in a table etc.




2. Adding a system call to xv6. How are sys calls implemented?

Here's the files changed: date.c syscall.c syscall.h sysproc.c user.h usys.S

In user.h: +int date(struct rtcdate*).

Actual implementation in sysproc.

Tracing a system call. Hm.

