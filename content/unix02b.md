+++
date = "2015-07-07T19:52:26+02:00"
draft = true
title = "What's on the stack?"

+++

This is the third post in my series on Grokking xv6. We will look at system
calls and how they work under the hood.

<!--more-->

## Introduction

Let's begin with a word of warning: in this article there'll be a lot
of things that you'll see that we won't explain. Instead, we are going
to learn how to squint at the code, suppress detail and focus on what
we want to do.

Hardest part for me was not having a good mental model of the stack
and how it operates, will attempt to give this to you.

TODO: Obsolete? More like one now

Three purposes:
1. See how a system call flows through the system.
2. Learn how to use GDB to figure such things out.
3. Learn about the stack.

## Code

We are going to look at some assembly. I do not expect you to
understand everything, instead we will look at a few things and see
what they can teach us about the stack.

Here's the C code for the main function in `ls.c`:

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

This is the main entry point of `ls`. `argc` stands for *argument
count*, and if main doesn't get more than two arguments (say, if you
write `ls foo/bar`) it will call the function `ls` with the argument
".", and after that it will call the `exit` function.

We can get the assembly for our main function with the `disassemble`
command in GDB:

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

This is a lot of code, and we are not going to understand all of
it. Here's how to read it. In the first line, `=>` means that this is
the instruction we are about to execute, but haven't yet. `0x00000304`
is the address of the instruction written in *base 16* or
*hexadecimal* or *hex* - this is where instruction lives in
memory. Another way of writing the above address is to omit the
leading zeros, so simply `0x304` (0x means it's a hexadecimal
number). `<+0>` is an offset that we are going to ignore. `push` is a
*mnemonic* or an *instruction*, and its argument in this case is
`%ebp`.

Without knowing anything about assembly, you might notice that the
`call` instruction occurs several times, and that one of its arguments
is `ls` and `exit`. This corresponds to the four function calls we see
in our main function.

## The Stack

When we write programs, we don't usually write a series of
instructions for the computer to execute. Instead, we use *structured
programming* and *subroutines* that call each other and can be used
multiple times. If we follow the execution of a program by pointing at
the screen we might say "this calls this, which calls this, then it
returns here". The stack is how the computer keeps track of things
like this, where it came from, what the argument of a function are,
etc.

Let's have a look at what's on the stack before we have executed a
single instruction. We can do this using GDB's `x` command, which
allows us to inspect memory at a given address. It takes two
arguments: a format and an address.

```
(gdb) x /x $esp
0x2fe8: 0xffffffff
```

This means that $esp is set to `0x2fe8` and the content of it is
`0xffffffff`. This is what's on top of the stack. We can see a bit
more of what's on the stack with the following.

```
(gdb) x /4x $esp
0x2fe8: 0xffffffff      0x00000001      0x00002ff4      0x00002ffc
```

In this case, the format `4x` means "show me 4 words in hex". A word
is 4 bytes long. A byte is 8 bits long and 8 times 4 is 32, which is
how big are our addresses and registers are. This makes sense, since
we are running on a 32-bit architecture. `$esp` is a special
*register* called *(extended) stack pointer*. Registers store data for
the processor for easy access.

At any given time, the stack pointer points to the top of the
*stack*. In the above output, `0x2fe8` is the value of $esp, and it
points to `0xffffffff`. We can also write the above in a slighlty more
verbose way, for clarity:

```
0x2fe8: 0xffffffff <--- ESP
0x2fec: 0x00000001
0x2ff0: 0x00002ff4
0x2ff4: 0x00002ffc
```

There's some confusion about what a stack is. In the abstract sense,
it's a *LIFO* (last in first out) data structure that supports two
primary operations: *push* and *pop*. What might be confusing is that
it's normally conceptualized as a stack of plates, growing upwards. In
this context, the stack grows down (in terms of addresses) but the
last touched entry is still called the top of the stack. Notice that
the top of the stack has the lowest address. This might be
counterintuitive, but "top" is an abstract term and it doesn't become
less of a stack because it's growing down.

If those four addresses show what's on the stack before we have even
executed a single instruction in our main function, what do they mean?
To understand that we have to talk about *calling
conventions*. Different programming languages have different calling
convention, and sometimes it can even differ on different computer
architecture. If you use C, most of them are quite similar though,
with minor differences (for example, if use certain compiler
optimizations). First we push the arguments of the function, in
reverse order, and then we push the return address, or the return PC
(program counter). Calling conventions cover a lot more than this, and
we will see some more usese of this in later sections.

What does this mean in our case? Our main function takes two
arguments, argc and argv. We can see what the value of those are:


```
0x2fe8: 0xffffffff    <- -1, fake return address for main
0x2fec: 0x00000001    <- 1, argc value 
0x2ff0: 0x00002ff4    <- argv, pointer to argv[0]
0x2ff4: 0x00002ffc    <- "ls", value of argv[0]
```

Since we know the value of argv[0] will be in that position, we can
cast it to a char* (string) to see its value. Likewise, we can do the
same for the fake main return address:

```
(gdb) print (char*)0x00002ffc
$83 = 0x2ffc "ls"
(gdb) print (int)0xffffffff
$88 = -1
```

Note that main is a bit special since it has no place to return to,
hence its "return address" is just -1. If you are confused about how
that number can be -1, here's what it looks like in binary using the
format `\t`, which in two's complement is -1. If it is -1, we would
expect it to be equal to 0 when we add 1 to it. And indeed:

```
(gdb) print /t 0xffffffff
$35 = 11111111111111111111111111111111
(gdb) print 0xffffffff + 1
$36 = 0
```

## One instruction at a time

We are now going to step through the beginning of the program,
instruction by instruction. We can see the first four instruction that
are about to be executed by running the following.

```
(gdb) x /4i $eip
=> 0x304 <main>:        push   %ebp
   0x305 <main+1>:      mov    %esp,%ebp
   0x307 <main+3>:      and    $0xfffffff0,%esp
   0x30a <main+6>:      sub    $0x20,%esp
```

`$eip` is a register that's called an *(extend) instruction
pointer*. This tells us where we are executing right now. The format
we are using interprets the next 4 words in memory as
instructions. There are no guardrails here - you have to be careful
about if you are casting an int to a char*, or if you read addresses
as instructions, etc. Sometimes you get an error message from GDB, but
far from all the time.

These four instructions are called the *function prologue*. What does
the stack look like after we perform these instructions?

Let's do it one by one using GDB's `stepi` function. After `push %ebp`
this is the stack:

```
0x2fe4: 0x00003fb8  <--- ESP
0x2fe8: 0xffffffff
0x2fec: 0x00000001
0x2ff0: 0x00002ff4
```

Note that two things have happened: we have moved the stack pointer up
(or down, in terms of memory location) an address, and put a new value
in there. The push instruction does both of those things one after
another. This is slightly tricky as there's no mention of $esp in that
instruction, but yet it changed our stack pointer. There are only a
few instructions like these though: `push`, `pop`, `call`, `ret` and a
few others. Another way to write `push $ebp` is as follows:

```
sub $0x4 $esp
mov $ebp ($esp)
```

This subtracts 4 from the stack pointer and puts $ebp in whatever the
stack pointer is pointing to. $esp is the address of the stack
pointer, and ($esp) is what the stack pointer points to.

After `mov %esp, %ebp` nothing changes. This moves our stack pointer
into the *base pointer* address.

What is $ebp? It stands for *(extended) base pointer* (or sometimes
it's called a frame pointer, depending on the architecture). The idea
is that the stack pointer changes throughout the function as variables
and registers are pushed and popped, but when the function returns we
can trust that the base pointer stays the same. This means that we
can, when the function is about to return, restore our stack pointer
to the base pointer, and thus we have easy access to the return
address, which is right below us! Note that the base pointer isn't
strictly necessary, since we (or rather, our compiler) can do this
arithmetic itself, but it can be convenient.

Let's move on. The next instruction is `and $0xfffffff0,%esp`, and
it's a so called stack alignment (`0xfffffff0` is -16 in hex, using
two's complement). It ensures that the stack pointer will be at its
current position in memory, or at a lower one, but more importantly
that it will be at a 16-byte boundary. Why this is done is outside of
the scope of this article, but it has to do with performance and being
able to do several instructions in parallel on certain architectures
like *SIMD*. We can see that the stack pointer is evenly divided by 16
with `print 0x2fe0 % 16` (or just by looking at the last position in
the hex number, since that's the "16"-th position). This is what the
stack looks like now:

```
0x2fe0: 0x00000000  <--- ESP
0x2fe4: 0x00003fb8
0x2fe8: 0xffffffff
0x2fec: 0x00000001
```

After `sub $0x20,%esp`, which subtracts `0x20` (2 times 16 is 32 in
decimal) from our stack pointer, we've made room for 32 bytes on the
stack. This is used for local variables in main.

```
(gdb) x /12x $esp
0x2fc0: 0x00000000      0x00000000      0x00000000      0x00000000
0x2fd0: 0x00000000      0x00000000      0x00000000      0x00000000
0x2fe0: 0x00000000      0x00003fb8      0xffffffff      0x00000001
```

We will show the stack using the more concise format from now on.

We've now performed four instructions in main and this is the end of
the *function prologue*. Something similar to this is done in every
function. There's also an *function epilogue* - and of course the main
meat of the function in between.

## Calling ls

If we step two more instructions (`stepi 2`) we get to:

```
movl   $0xb5f,(%esp)
```

which puts 0xb5f on the stack without changing the stack pointer (good
thing we made room for local variables before so we didn't overwrite
our stack!) What is that? Just like was done to `main` in the very
beginning, before we call the `ls` (the function, not the program) we
push the argument on the stack. We know it should be a string, and we
can see the true nature of it by casting it to a char*:

```
print (char*)0xb5f
$2 = 0xb5f "."
(gdb) x /4x $esp
0x2fc0: 0x00000b5f      0x00000000      0x00000000      0x00000000
```

The next instruction is `call 0xb0 <ls>`. `call` does two things, it
pushes the address of the next instruction onto the stack, and then it
jumps to a subprogram (which is just another address in memory, but
since our program was compiled with debug information on we can see
that it says <ls>). After executing that instruction our stack and our
upcoming four instructions to execute looks like this:

```
(gdb) x /4x $esp
0x2fbc: 0x0000031f      0x00000b5f      0x00000000      0x00000000
(gdb) x /4i $eip
=> 0xb0 <ls>:   push   %ebp
   0xb1 <ls+1>: mov    %esp,%ebp
   0xb3 <ls+3>: push   %edi
   0xb4 <ls+4>: push   %esi
```

We are now in the `ls` function, and `0xb0`, which was an argument to
call, is what our instruction pointer is set to. You might recognize
the two first lines from the prologue in our main function.

What about `0x0000031f` that's now on top of our stack? It's the
address where the program should keep executing oncs the ls function
returns. We can confirm this by looking at it's memory location as
instructions. These are exactly the instructions that come after the
call to ls (see the code section above).

```
(gdb) x /4i 0x0000031f
   0x31f <main+27>:     call   0x5cb <exit>
   0x324 <main+32>:     movl   $0x1,0x1c(%esp)
   0x32c <main+40>:     jmp    0x34d <main+73>
   0x32e <main+42>:     mov    0x1c(%esp),%eax
```

Let's step two more instructions, pushing the base pointer onto the
stack and moving (or "saving") the stack pointer to the base pointer.

```
(gdb) x /4x $esp
0x2fb8: 0x00002fe4      0x0000031f      0x00000b5f      0x00000000
```

We got a new address on the stack, which was our old base pointer from
main. We've seen this before, but let's take a look again.

TODO: this part is confusing, see if it clears up after skipping in ls

```
(gdb) x /4x 0x00002fe4
0x2fe4: 0x00003fb8      0xffffffff      0x00000001      0x00002ff4
```

This is our stack at the beginning of main.

## Skipping until the end of ls

There's a lot that happens in `ls` and the functions it calls. We are
going to skip all that by going to the very last line of the C code in
the ls function, line 71. We do this with `until 71`. What's about to
happen now and what does the stack look like?

```
(gdb) x /6i $eip
=> 0x2f9 <ls+585>:      add    $0x25c,%esp
   0x2ff <ls+591>:      pop    %ebx
   0x300 <ls+592>:      pop    %esi
   0x301 <ls+593>:      pop    %edi
   0x302 <ls+594>:      pop    %ebp
   0x303 <ls+595>:      ret
(gdb) x /4x $esp
0x2d50: 0x00000003      0x00002d88      0x00000010      0x00000001
```

I don't know what those things on the stack are, but presumably they
come from something ls did - in fact, we would have to run `x /170x
$esp` to begin to see addresses that we recognize on our stack. That's
okay though, we are about the clean that stack up with the function
epilogue. Essentially the five first instructions are the opposite of
subtracting and pushing things onto the stack. If we step through them
with `stepi 5` we get:

```
(gdb) x /4x $esp
0x2fbc: 0x0000031f      0x00000b5f      0x00000000      0x00000000
```

Before we did that our stack pointer was set to `0x2d50`, and now it's
back to `0x2fbc`. As a sanity check on what's going on, we can see
that everything seems OK: we just cleaned up 620 bytes, of which the
first `add $0x25c, %esp` took care of 604. `pop` was called four
times, and 4 times 4 bytes (the size of each register that we popped)
is 16, which takes care of the rest.

```
print  0x2fbc - 0x2d50
$3 = 620
print 0x25c
$4 = 604
```

Our stack pointer is now back at `02fbc`, and the next instruction is
a simple `ret`. What does `ret` do? It's the opposite of `call`: it
pops off an address, and jumps to it. So without single stepping, we
should expect it to jump to `0x0000031f` and set the stack pointer to
`0x2fbc - 4` = `0x2fb8`. What is `0x31f` again? It's the next line in
`main` after calling `ls`. Let's single step it:

```
x /4x $esp
0x2fc0: 0x00000b5f      0x00000000      0x00000000      0x00000000
x /4i $eip
=> 0x31f <main+27>:     call   0x5cb <exit>
   0x324 <main+32>:     movl   $0x1,0x1c(%esp)
   0x32c <main+40>:     jmp    0x34d <main+73>
   0x32e <main+42>:     mov    0x1c(%esp),%eax
```

Indeed, we are now back in `main` and are about to call `exit`, and
our stack is back to the same state it was just before it was about to
call `ls`.
