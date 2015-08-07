+++
date = "2015-08-06T20:24:36+02:00"
draft = true
title = "GROK LOC?"

+++

> (repeatedly 20 #(rand-int 10000))

Rules: no empty lines or lines that {} or comments etc

Few Q: Would you write this line yourself? Do you understand why it is
there?

As a rule, comments (ie no influence code) and lines you'd find in
other code bases, like "static int"

Also, immediately. Not after thinking a lot and doing a lot of
googling. INSTA.

Alt can provide context, paragraph. Wordy though.

20 RANDOMS, #1
13 n/a - more than half!!!

<pre>
Aug 6 2015

1
0354 int pipewrite(struct pipe*, char*, int);
CONTEXT: defs.h: declarations for pipe.c
COMMENT: Pipe interface declaration to write to a pipe.
VERDICT: YES

2
3093 acquire(&kmem.lock);
CONTEXT: kalloc.c: kalloc function
COMMENT: Get lock so we don't get lost update in memory's free list.
VERDICT: YES

3
4256 struct buf **pp;
CONTEXT: ide.c: iderw function
COMMENT: Temp buffer for disk sync, don't know why ** and purpose.
VERDICT: NO

4
5525 if((next = dirlookup(ip, name, 0)) == 0){
CONTEXT: fs.c: namex function
COMMENT: Haven't studied the file system yet. (Aug 6)
VERDICT: NO

5
6144 f−>off = 0;
CONTEXT: sysfile.c: sys_open function
COMMENT: Don't know what off is or why we set it to 0. See above.
VERDICT: NO

6
6638 if(s < d && s + n > d){
CONTEXT: string.c: memmove function
COMMENT: Don't know immediately. Making sure we move memory correctly.
VERDICT: NO

7
8331 exit();
CONTEXT: init.c: main function, when pid == 0
COMMENT: Path only reached when exec of ls fails, exits init process.
VERDICT: YES

8
0361 int growproc(int);
CONTEXT: defs.h: proc.c declarations
COMMENT: Declaration for process API to grow a process's memory.
VERDICT: YES

9
0668 #define STA_W 0x2 // Writeable (non−executable segments)
CONTEXT: asm.h: magic constants.
COMMENT: I don't know. Some segment type bit for x86.
VERDICT: NO

10
1912 memmove(mem, init, sz);
CONTEXT: vm.c: inituvm function
COMMENT: Move memory char* in init to the allocated mem-location mem.
VERDICT: YES

11
5078 if(ip−>type == 0)
CONTEXT: fs.c: ilock function
COMMENT: Can't lock inode, unexpected that inode has no type.
VERDICT: YES

12
5523 return ip;
CONTEXT: fs.c: namex function
COMMENT: Found inode for a path name and returns it.
VERDICT: YES

13
5613 struct spinlock lock;
CONTEXT: file.c: in struct ftable
COMMENT: A lock for accessing file table.
VERDICT: YES

14
0273 int exec(char*, char**);
CONTEXT: defs.h: declarations for exec
COMMENT: For executing procs, takes a path and pointer to argv string.
VERDICT: YES

15
0531 pd[1] = (uint)p;
CONTEXT: x86.h: lidt function
COMMENT: I don't know, used for some inline assembly lidt.
VERDICT: NO

16
0709 #define FL_TF 0x00000100 // Trap Flag
CONTEXT: mmu.h: eflags register magic constants.
COMMENT: I don't know. Constant to signify if it's a trap to x86?
VERDICT: NO

17
3403 cprintf("unexpected trap %d from cpu %d eip %x (cr2=0x%x)\n",
CONTEXT: trap.c: trap switch statement, default case
COMMENT: Got an unknown trap from kernel space, prints debug info.
VERDICT: YES

18
6240 if(uarg == 0){
CONTEXT: sysfile.c: sys_exec function
COMMENT: If current, temp, arg is 0 we can pad argv to zero.
VERDICT: YES

19
7482 if(irqmask != 0xFFFF)
CONTEXT: picirq.c: picinit function
COMMENT: Something about interrupt controller missing a bit mask.
VERDICT: NO

20
7819 for(;;)
CONTEXT: console.c: panic
COMMENT: Infinite loop after kernel panic.
VERDICT: YES

RESULT: 12/20
</pre>


Really interesting. Keep doing until you have 20.

GROKTESTS.
~$ rrange 0.95 20
Around 19 ~ [17, 21]

So 17+ is OK.

what does it mean to understand a line of code?
