+++
date = "2015-06-26T19:00:00+02:00"
title = "What is a shell and how does it work?"
+++

This post is part of my ongoing experiment in grokking xv6. In it I will teach
you what a shell is and how it works.

<!--more-->

If you haven't read first part in this series, you can read it
[here](http://experiments.oskarth.com/unix00/).

In my experience, most technical explanations are full of jargon and implicit
knowledge. Throughout the text I've deliberately italicized the first use of all
terms that can be seen as domain-specific. In cases where I think an elaboration
seems necessary, I've included either a short in-line explanation or a footnote.
Does this type of explanation - being explicit about the words you use - help
with understanding, or is the italicization just getting in the way of reading?

In general, any explanation that is clear and concise is good. This type of
explanation tries to err more on the side of clarity, rather than conciseness. [^1]

## What is a shell?

A *shell* is just another *computer program*. It is the main *user interface* to
*operating systems* that are similar to *Unix*. An operating system is
responsible for having several programs run on one *computer*, and also to tries
to *abstract away* the specific *hardware* the computer is running on, so the
same program can run on many different types of computers. Unix is a special
type of operating system that was developed at *Bell Labs* by *programmers* like
*Ken Thompson*, *Dennis Ritchie*, and *Brian Kernighan*. A *kernel* is the part
of the operating system that we will be concerned with. Modern operating systems
also provide things like *graphical interfaces*.

The shell lives in *user space*, along with most other programs, as opposed to
in *kernel space* which is where the kernel lives. *Software* living in kernel
space can execute *privileged instructions*, such as dealing directly with
hardware. We don’t want any software to be able to do this, as it could
*overwrite* the operating system itself. The way the shell talks to the kernel
is by system calls [^2]. These system calls allows the user to do things like
*open files* and *create processes*. Since software in user space always have to
go through the kernel to perform such operations, the kernel can make sure the
shell doesn't do anything it doesn't want to allow. Note that this is different
from a *super-user* or *running as root*, which is about *user privilege* that
software in user space have.

## Hello shell

If you want to see a list of what's inside the *directory* in your *present
working directory* [^3] you can use the program *ls*. Typing `ls` in a shell and
pressing enter runs it and returns the contents of that folder. When you press
enter the shell first *parses* [^4] what you wrote into some internal
representation. What does it mean to run `ls`? To answer that we must first
understand what the *fork* system call does and what a *child process* is.

Recall that an operating systems allows several programs to run on one computer,
and processes are a big part of how it does that. A *process* in Unix-like
systems is a program that runs and has access to its own piece of *memory* which
contains the program's *instructions*, *data* and *stack*. The operating system
then makes sure each process gets to do what it wants to do in some reasonable
manner. Fork is a system call that allows a process to create another process.
The process *calling* fork is called the parent process, and the process it
creates is called the child process. When a child process starts, its memory is
initally almost an exact, but separate, copy of its parent process's memory.

In the case of our `ls` command, the shell runs the parsed command in a forked
child process. In *C* *code* it would look something like this:

    int pid = fork();
    if(pid == 0)
      runcmd(parsecmd(buf));
    wait();

When we call fork we create a child process, and we get back an integer that we
call *pid* for process id. We now have two different processes running
independently, and the *variable* pid is different in the two. In the parent
process, the pid is some number that is used to uniquely identify the process,
whereas in the child process it's simply equal to 0.

A good way to think about the above piece of code is to see it as being two
different programs. In the parent process, the *if-statement* returns false and
it gets stuck at *wait*, which is another system call that just waits until a
child process is finished. The kernel makes sure the parent is notified when
that happens. In the child process, the if-statement returns true and it runs
the command. This executes `ls` and gives over all control to `ls` for that
process.

Once the child process has finished running - and when that happens is
completely up to `ls` and how it is implemented - the parent process, i.e. the
shell, will resume running, and we can type another command.

## I/O Redirection

Let's say we want to save the *output* from running `ls` above. We can do so
using something called *I/O redirection* (I/O stands for *input/output*):

    ls > foo

There are three parts to this command. When the shell parses the command, it
figures out that it's a redirection from `ls` to `foo`. After the shell has
forked to a child process it runs `ls` and saves the output in a *file* called
foo. An ordinary file contains either *symbolic* or *binary* data, is written in
some *format*, and has some *metadata* associated with it (such as who is
allowed to read and write to it), and we can access it by using its *pathname*.
One of the most common type of file is a *text file*, which contains a *string*
of *characters*. There are special files too, and in fact even *devices* and
*directories* are represented as files.

How does the output of ls end up in foo? To understand that we have to know a
bit more about files work in Unix. The system call *open* is used to see the
contents of a file. When we open a file to read or write to it, we get a *file
descriptor* back. This file descriptor is just an *integer* that represents a
specific file that a process can read or write to. There are three special
integers, 0, 1, and 2. These are called, in order, *STDIN* (standard input),
*STDOUT* (standard output), and *STDERR* (standard error). Another word for
these is *standard streams*. By default, when the shell reads something, such as
a command you typed in, it does this from STDIN. Likewise, when the shell prints
something, such as the result of running some command, it does this to STDOUT.

There is some confusion about the difference between files and streams, and
people can mean different things when they talk about them. For our present
purposes, we can treat them as equivalent - as long we let go of our
preconceptions of what a file is. As alluded to before, many things are seen as
as files from the kernel's point of view in Unix-like systems.

When we run `ls`, it returns the result by *printing* it [^5] to STDOUT, and
STDOUT is what we see in the shell. File descriptors are handed out by the
kernel starting from the lowest available file descriptor, and when the shell
starts it opens the three standard streams. We can use this fact to get I/O
redirection with something like this:

    close(fd);
    open(file, mode);
    runcmd(cmd);

This is a simplified version of the actual code and it has no error handling.
The second *argument* to open is mode, which is where we say if we want to read
or write to the file. Assuming we want to redirect using `>`, we close STDOUT.
When we then open the file, foo in this case, to write to it, it will pick the
lowest available file descriptor, which is 1. When we then run the command `ls`,
it will print to STDOUT - which is *bound* to our file foo.

## Pipes

Let's look at another example. There's a program called *ps* that shows the
*status* of processes. If we want to have a list of all my processes sorted by
their process id we can *pipe* the result of running ps to the program *sort*.

    ps | sort

When the shell parses this command it sees the symbol "|" and knows
it's a pipe command. A pipe command has two sides: a left and a right
side. When the program on the left side writes to STDOUT it can be
read from the program on the right side through STDIN. A pipe is a
small *buffer* [^6] that lives in kernel space and allows processes to
talk to each other, which is called *inter-process
communication*. This communication happens continuously as new data is
written to the pipe. It's also a *queue*, so even if new data comes in
faster than you can process it in the right process the data doesn't
disappear, and it doesn't *block* [^7] either.

How does this work? Let's look at the code.

    int p[2];
    pipe(p);

    if(fork() == 0) {
      close(1);
      dup(p[1]);
      close(p[0]);
      close(p[1]);
      runcmd(left);
    }
    if(fork() == 0) {
      close(0);
      dup(p[0]);
      close(p[0]);
      close(p[1]);
      runcmd(right);
    }
    close(p[0]);
    close(p[1]);
    wait();
    wait();

We create an array of two integers, which is where we will keep track of our
file descriptors. We then use the system call pipe, which creates a pipe between
two file descriptors and and puts these in p, where p[0] is for reading and
p\[1] is for writing. After that, we create two child processes - one for the
left process and one for the right one. These are the two if-blocks that check
if fork returns 0, which it does in the child processes.

In the left process we close STDOUT. Then we use another system call *dup* that
duplicates a file descriptor. What this means is that we can refer to the same
file or stream but using a different file descriptor. Since STDOUT is closed
and it's the lowest available file descriptor, the write-end of our pipe gets
connected to STDOUT in this process.

Recall that a child process has almost exactly the same memory as a parent
process. This includes file descriptors, so after we have connected the left
process to STDOUT we want to close the file descriptors in p. If we don't, we
might get a *deadlock* where we don't ever see anything printed. For example, if
we forget to close the write-end of the pipe (p\[1]) in the right child process,
the read-end of the pipe (p[0]) will keep waiting for data. This means that the
left child process won't ever finish, and the the parent process will wait
forever. It's only when the write-end of a pipe is closed that the read-end
stops waiting. This is similar to how ordinary files have a specific
*end-of-file* character that tells us when a file is finished.

Depending on the exact command, things might still work out fine even if you
forget to close a file descriptor. However, in order to avoid subtle bugs,
consider it good hygiene to close a file descriptor when you are done with it.

After all that, we run the left process and it prints to STDOUT. Similarly, the
right process does almost the exact same thing but for STDIN. And finally, the
parent closes the file descriptors for the pipe and waits for both child
processes to finish (which, if it's a *long-running process*, may never
happen).

## Conclusion

In this article we've looked at how the shell executes a simple command, how I/O
redirection works, and how pipes allow for inter-process comunication. We've
also looked briefly at what files and process are, and how to use some basic
system calls.

I hope I have managed to paint a clear picture of the shell. If you wish to
solidify your understanding of the shell, I recommend you to do what I did and
go through the xv6 shell in detail, and perhaps do some of the suggested
homework in MIT's operating systems class. If you have any comments, please
don't hesitate to [email](mailto:me@oskarth.com) or
[tweet](https://twitter.com/oskarth) me.

**Thanks** to Mark Dominus, Margo Kulkarni, and Kamal Marhubi for
  reading drafts of this post.

(If you liked this, you might enjoy
[What's on the stack?](http://experiments.oskarth.com/unix02/). To
stay up to date on my experiments, consider
[subscribing](http://eepurl.com/bvtdfj).)

## Resources

- xv6 shell code: https://github.com/oskarth/xv6/blob/master/homework/sh.c
- MIT's OS class: http://pdos.csail.mit.edu/6.828/2014/
- xv6 book: http://pdos.csail.mit.edu/6.828/2014/xv6/book-rev8.pdf
- xv6 code: http://pdos.csail.mit.edu/6.828/2014/xv6/xv6-rev8.pdf
- Unix paper: http://www.cs.berkeley.edu/~brewer/cs262/unix.pdf

[^1]: But conciseness is still more important than completeness.

[^2]: *System calls*: Things you can tell the kernel to do in kernel space.

[^3]: *Present working directory*: Where you are in the *file hierarchy* at any given time.

[^4]: *Parsing*: Figures out the parts of what you told it are.

[^5]: *Printing*: Writing it in a place where you can see it.

[^6]: *Buffer*: A place where temporary data is stored.

[^7]: *Block*: Get stuck waiting.
