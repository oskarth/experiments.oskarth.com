+++
date = "2015-07-07T19:52:26+02:00"
draft = true
title = "What's on the stack?"

+++

## Setup - delete this section, not a tutorial

In one terminal, which is where we will run xv6 using the emulator
QEMU, we type `make qemu-nox-gdb`.

In another, which is where we will use GDB to debug and look at the
memory. I start `gdb` through emacs, but in a terminal window works
just as fine. After that we run the following inside GDB to start
debugging and setting a *breakpoint* at the main function.

```
(gdb) symbol-file _ls
Reading symbols from _ls...done.
(gdb) break main
Breakpoint 1 at 0x304: file ls.c, line 75.
(gdb) continue
Continuing.
```

Lines that start with (gdb) are lines that we type in. GDB is now
waiting for something to happen. We go back to our first window and
type `ls`. Note that it doesn't show any results, that's because our
breakpoint halted the execution at our breakpoint.

## Code

We are going to look at some assembly. I do not expect you to
understand everything, instead we will look at a few things and see
what they can teach us about the stack.

Here's the C code for main:

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
memory. `<+0>` is an offset that we are going to ignore. `push` is a
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
(gdb) x /4x $esp
0x2fe8: 0xffffffff      0x00000001      0x00002ff4      0x00002ffc
```

In this case, the format `4x` means "show me 4 words in hex". A word
is 4 bytes long. A byte is 8 bits long and 8 times 4 is 32, which is
how big are our addresses and registers are. This makes sense, since
we are running on a 32-bit architecure. `$esp` is a special *register*
called *(extended) stack pointer*. Registers store data for the
processor for easy access.

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
with minor differences. First we push the arguments of the function,
in reverse order, and then we push the return address. Here's how it
might look in the abstract case:


What does this mean in our case? Our main function takes two
arguments, argc and argv. We can see what the value of those are:


TODO: This bit is confusing

```
(gdb) print argv
$81 = (char **) 0x2ff4        // pointer to a char* (string)
(gdb) print *arg v            // or print argv[0]
$82 = 0x2ffc "ls"
(gdb) print (char*)0x00002ffc // or, iff we just have memory location
$83 = 0x2ffc "ls"
(gdb) print argc
$89 = 1
(gdb) print (int)0xffffffff
$88 = -1
```

Since main is a bit special, its "return address" is just -1.

## Stepping one instruction at a time

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
pointer*. This tells us where we are right executing right now. The
format we are using interprets the next 4 words in memory as
instructions. There are no guardrails here - you have to be careful
about if you are casting an int to a char*, or if you read addresses
as instructions, etc. Sometimes you get an error message from GDB, but
far from all the time.

These four instructions are called the *function prologue*. What does
the stack look like after we do this?

TODO: explain this better

Let's do it one by one. After `push %ebp` this is the stack:

```
0x2fe4: 0x00003fb8  <--- ESP
0x2fe8: 0xffffffff
0x2fec: 0x00000001
0x2ff0: 0x00002ff4
```

After `mov %esp, %ebp` nothing changes. This moves our stack pointer
into the *base pointer* address.

TODO: explain that better

After the next `and` instruction which is a stack alignment
(`0xfffffff0` is -16 in hex, using two's complement).

```
0x2fe0: 0x00000000  <--- ESP
0x2fe4: 0x00003fb8
0x2fe8: 0xffffffff
0x2fec: 0x00000001
```

After `sub $0x20,%esp`, which subtracts `0x20` (32 in decimal) from
our stack pointer which makes room for 32 bytes on the stack. This is
used for local variables in main.

```
0x2fc0: 0x00000000
...                 // seven more addresses full of zeroes
0x2fe0: 0x00000000
0x2fe4: 0x00003fb8
```

TODO: WHO CARES?

This is the end of the prologue. If we step two more instructions we
get to:

0x00000313 <+15>:    movl   $0xb5f,(%esp)

which puts 0xb5f on the stack without changing the stack pointer. What
is that? Before we call the `ls` function we push the argument on the
stack. We can see the true nature of that by casting it to a char*:

```
print (char*)0xb5f
$105 = 0xb5f "."
(gdb) x /4x $esp
0x2fc0: 0x00000b5f      0x00000000      0x00000000      0x00000000
```

The next instruction is `call 0xb0 <ls>`.

```
(gdb) x /4x $esp
0x2fbc: 0x0000031f      0x00000b5f      0x00000000      0x00000000
```

There's a new address on top. Looks like call modified the stack. It
did two things, one jump to 0xb0, which is where ls lives, and two
push 0x0000031f onto the stack. What is 0x0000031f? It's where the
program should keep executing ones the ls function is done. We can
confirm this by looking at it's memory location as instructions. These
are exactly the instructions that come after the call to ls.

```
(gdb) x /4i 0x0000031f
   0x31f <main+27>:     call   0x5cb <exit>
   0x324 <main+32>:     movl   $0x1,0x1c(%esp)
   0x32c <main+40>:     jmp    0x34d <main+73>
   0x32e <main+42>:     mov    0x1c(%esp),%eax
```

Now we are in `ls`.


You don't really explain things, O. Clarify what you want to explain!

## QUESTIONS/NOTES FOR AUTHOR

Lajout:

address: content ; comment

pushin' and poppin'.

