+++
date = "2015-05-29T19:00:00+02:00"
title = "Nand to Tetris 2"
+++

This post is the second part of our Nand to Tetris mental model
diffing series. This week we look at machine language, computer
architecture, and assemblers.

<!--more-->

The format is the same as
[last week](http://experiments.oskarth.com/nand-to-tetris-1/) - I've
written down a collection of assertions and then received answers and
comments from people who know a lot more about these things. Several
people provided feedback this this time and you'll find a list of them
at the end of this post. Let's begin!

## Assertions

**1. Assembly language is just mnemonic sugar over binary codes.**

This means that if you read assembly you can straightforwardly, albeit
tediously, translate it into binary code. It also means you have “one
assembler” per computer architecture.

*There are certainly "high level" assembly languages where one
mnemonic has multiple possible valid translations, but in general it's
accurate. It depends on what you mean exactly, for example look at
AT&T vs Intel x86 assembly - they’re quite similar, and can be
translated to each other, but people get really annoyed when they’re
used to one and have to deal with the other, even just reading
it. (And all of this, in turn, is leaving out what defines a computer
architecture - is a system that can be big-Endian or little-Endian one
architecture or two?)*

**2. Screen, Keyboard et al. are usually accessed through a memory map.**

Alternatively, I know we can issue high-level instructions to a GPU
that yields much better performance, such as "draw a line from here to
here". I don't know why that is.

*This depends on both the hardware and the OS, but I think that this
is generally not true for "slow" I/O devices like a keyboard. For
example, in x86/DOS, I believe that you get an
[int 0x09](http://webpages.charter.net/danrollins/techhelp/0106.HTM)
for each keypress. With a modern version of Windows and an x64/SMP
system, it somehow uses ACPI and APIC, and I believe it’s valid to
think of that as an abstraction over
[interrupts](https://en.wikipedia.org/wiki/Interrupt). The DOS case is
pretty simple, and the link above explains how you can get access to
keypresses. Modern interrupt I/O is much more complicated, and for
Windows you can read more about it in
[Russinovich’s Windows Internals book](http://www.amazon.com/Windows-Internals-Part-Developer-Reference/dp/0735648735/).*

*Fast I/O devices will just
[DMA](https://en.wikipedia.org/wiki/Direct_memory_access) stuff in and
out of memory, though. There are multiple reasons why fast I/O devices
are faster. The first is that they can just directly talk to memory
instead of having to talk to a device, which talks to another device,
which signals the CPU to talk to the I/O device to do work. And that
signal will cause a context switch,
[which is pretty expensive](http://danluu.com/new-cpu-features/#context-switches-syscalls).*

*Another reason is that they can sit on much faster busses. PCIe gen 3
can deliver about 1GB per lane, and it’s not uncommon to see 16+ lane
devices. That’s a lot more than you can get out of USB. Compare that
to USB3, where you get maybe 10Gb/s, i.e., barely more than 1GB/s,
total. And you can do even better than PCIe if you’re on the same bus
that the processor is on, and that’s one reason that AMD and IBM
expose that stuff. I don’t know if you can get a
[QPI](https://en.wikipedia.org/wiki/Intel_QuickPath_Interconnect)
license from Intel, but maybe?*

**Why is the GPU so much faster?**

*Using a memory map adds memory copy latency and saturates the memory
bandwidth between CPU and GPU.  For example, consider pressing return
to create a new line at the bottom of a page -- you need to scroll
everything on the view up by a line height, then add your new blank
line.  Doing this pixel by pixel is slower than issuing a GPU command
that says "copy a rectangle from [(x=0, y=100), (x=width, y=height)]
to [(x=0, y=0), (x=width, y=height-100)], then fill in a background
color area from height-100 to height."*

*In short: The GPU is so much faster because there’s no
throughput/latency bottleneck in copying memory across the
motherboard. GPUs are also optimized for parallel processing (such as
performing an operation on a matrix of pixels), and CPUs aren’t.*

**3. GOTOs and jumps are the only branching statements in Assembly.**

*This depends on the assembly language and architecture. For example,
x86 has a `loop` instruction. This is not just a macro. It’s possible
some CPUs will implement `loop` as a conditional branch, but it’s
literally a hardware instruction. For an interesting example, look up
x86 string instructions. They were designed to be fast, then were a
slow option that stuck around for backwards compatibility, etc.*

**What about Dijkstra's "GOTO considered harmful?"**

*Dijkstra wanted to promote another, more structured, way of dealing
with software: procedures with one entry and one exit point. Taken to
its extreme, this leads to hard-to-analyze programs, full of extra
control flags for the one exit point of the function, and much uglier
error-handling code - the Linux kernel extensively uses goto in
functions where there have to be multiple memory allocations, any of
which may fail, and where the subset that were done need to be cleaned
up, for instance. All that said, some level of structured programming
is generally much preferable to most uses of GOTO.*

**4. The von Neumann bottleneck is still a thing.**

Our computer architecture illustrates the
[von Neumann bottleneck](http://en.wikipedia.org/wiki/Von_Neumann_architecture#Von_Neumann_bottleneck),
without explicitly talking about it. Here's a cameo from
[John Backus](https://en.wikipedia.org/wiki/John_Backus) which laments
the existance of it:

*Surely there must be a less primitive way of making big changes in
the store than by pushing vast numbers of words back and forth through
the von Neumann bottleneck. Not only is this tube a literal bottleneck
for the data traffic of a problem, but, more importantly, it is an
intellectual bottleneck that has kept us tied to word-at-a-time
thinking instead of encouraging us to think in terms of the larger
conceptual units of the task at hand. Thus programming is basically
planning and detailing the enormous traffic of words through the von
Neumann bottleneck, and much of that traffic concerns not significant
data itself, but where to find it.* (from his
[Turing Award lecture](http://www.thocp.net/biographies/papers/backus_turingaward_lecture.pdf)).

*Yes. Higher level languages help conceptually, but not so much in
terms of implementation. Having tools to cut the CPU out of dealing
with things like blatting data from devices to RAM helps a bit. But
fundamentally, CPUs have gotten really fast, memory access is
amazingly slow, disk access slower yet, and caches only partly
help. [This](http://stackoverflow.com/questions/4087280/approximate-cost-to-access-various-caches-and-main-memory)
has some numbers that give an idea of the costs.*

**5. All symbols in an Assembler resolve to some memory address.**

*No, one might denote a numerical constant, for instance. Symbol
resolution is surprisingly deep, and parts happen at runtime - check
out LD_PRELOAD, LD_LIBRARY_PATH, and similar under Linux, for
instance, where even the library to be loaded, much less addresses
within it, can vary every time you launch the program, not just at
compile time. [Linkers and loaders](http://www.iecc.com/linker/) if
quite a fun read if you have a few hours to spare.*

## Conclusion

All the credit goes to these people, and any inaccuracies are due to
this author:

- [Dan Luu](http://danluu.com/)
- [Chris Ball](http://printf.net/)
- Darius Bacon
- Kat

Compared to last week, things got a lot more subtle quickly. It's not
that last week was less complicated, but the answers and diff seemed
more straightforward. I suspect this will be increasingly true as we
move up the stack, as things change more quickly the higher up we get.

A very low-resolution view of how the assertions fared would go
something like this: Ish, No, No, Yes, No. This is good news: it means
we are learning :)
