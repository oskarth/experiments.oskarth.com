+++
date = "2015-06-26T19:00:00+02:00"
title = "What is a shell and how does it work?"
+++

This post is part of my ongoing experiment in grokking xv6. In it I will teach
you what a shell is and how it works, using very simple language.

<!--more-->

If you haven't read first part in this series, you can read it
[here](http://experiments.oskarth.com/unix00/).

In my experience, most technical explanations are full of jargon and implicit
knowledge. Throughout the text I've deliberately italicized the first use of all
terms that can be seen as domain-specific. In cases where I think an elaboration
seems necessary or desirable, I've included either a short in-line explanation
or a footnote. Does this type of explanation - being explicit about the words
you use - help with understanding, or is the italicization just getting in the
way of reading?

In general, any explanation that is clear and concise is good. This type of
explanation tries to err more on the side of clarity, rather than conciseness.
[^1]

## What is a shell?

A *shell* is just another *computer program*. It is the main *user interface* to
*operating systems* that are similar to *Unix*. An operating system is
responsible for having several programs run on one *computer*, and also to tries
to *abstract away* [^2] the specific *hardware* the computer is running on, so
the same program can run on many different types of computers. Unix is a special
type of operating system that was developed at *Bell Labs* [^3] by *programmers*
[^4] like *Ken Thompson*, *Dennis Ritchie*, and *Brian Kernighan*. A *kernel* is
the part of the operating system that we will be concerned with. Modern
operating systems also provide things like *graphical interfaces*.

The shell lives in *user space*, along with most other programs, as opposed to
in *kernel space*. The way it talks to the kernel is by *system calls* [^5]
These system calls allows the user to do things like open *files* and create
*processes*.

## Hello shell

If you want to see a list of what's inside the *directory* [^6] in your *present
working directory* [^7] you can use the program *ls*. Typing `ls` in a shell and
pressing enter runs it and returns the contents of that folder. How does this
work? When you press enter the shell *parses* [^8] what you wrote and then, for
most commands anyway, runs that parsed *command* in a *forked* *child process*.

Let's take a step back. What's a fork? And what exactly is a process? Recall
that an operating systems allows several programs to run on one computer, and
processes are a big part of how it does that. A process in Unix-like systems is
a program that runs and has access to its own piece of *memory* which contains
the program's *instructions*, *data* and *stack*. The operating system then
makes sure each process gets to do what it wants to do in some reasonable
manner. Fork is a system call that allows a process to create another process.
The process *calling* fork is called the parent process, and the process it
creates is called the child process. When a child process starts, its memory is
initally almost an exact, but separate, copy of its parent process's memory.

In *C* [^9] *code* [^10] it would look something like this:

    int pid = fork();
    if(pid == 0)
      runcmd(parsecmd(buf));
    wait();

When we call fork we create a child process, and we get back a *pid*, or a
process id. This process id uniquely identifies that process. We now have two
different processes running independently, and the pid is different in the two.
In the parent process, the pid is some number that is used to identify the
process, whereas in the child process it's simply equal to 0.

A good way to think about the above piece to code is to see it as being two
different programs. In the parent process, the *if-statement* returns false and
it gets stuck at *wait*, which is another system call that just waits until a
child process is finished. The kernel makes sure the parent is notified when
that happens. In the child process, the if-statement returns true and it runs
the command. This executes `ls` and gives over all control to `ls` for that
process.

Once the child process has finished running - and when that happens is
completely up to `ls` and how it is implemented - the parent process, i.e. the
shell, will resume running, and we can type another command.

## Redirection

Let's say we want to save the *output* from running `ls` above. We can do so
using something called *input/output* *redirection*:

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
descriptor* back. This file descriptor is just an *integer*. There are three
special integers, 0, 1, and 2. These are called, in order, *STDIN*, *STDOUT*,
and *STDERR*. Another word for these is *standard streams*. Streams and files
are closely related - the difference is that a stream doesn't necessarily end,
since a process can keep writing to it while another process is reading from it.

When we run `ls`, it returns the result by *printing* it [^11] to STDOUT, and
STDOUT is what we see in the shell.  File descriptors are handed out by the
kernel starting from the lowest available file descriptor, and when we start the
shell all standard streams are already open.  We can use this fact to get I/O
redirection with something like this:

    close(fd);
    open(file, mode);
    runcmd(cmd);

Note that this is a simplified version of actual code and it has no error
handling. The second *argument* to open is mode, which is where we say if we
want to read or write to the file. Assuming we want to redirect using `>` (we
can also do it the other way using `<`), we close STDOUT. When we then open the
file, foo in this case, to write to it, it will pick the lowest available file
descriptor, which is 1. When we then run the command `ls`, it will print to
STDOUT - which is *bound* to our file foo.

## Pipes

Let's look at another example. There's a program called *ps* that shows the
*status* of processes. If we want to have a list of all my processes sorted by
their process id we can *pipe* the result of running ps to the program *sort*.

    ps | sort

When the shell parses this command it sees the symbol "|" and knows it's a pipe
command. A pipe command has two sides: a left and a right side. When we write on
the left side we can read from right side. A pipe is a small *buffer* [^12] that
lives in kernel space and allows processes to talk to each other, which is
called *inter-process communication*. This communication happens continuously as
new data is written to the pipe. If we write to a temporary file first before
running the second process the first process would have to finish before we the
second one starts. It's also a *queue*, so even if new data comes in faster than
you can process it in the right process the data doesn't disappear, and it
doesn't *block* [^13] either.

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

We create an array of two integers, which is where we will track of our file
descriptors. We then use the system call pipe, which creates a pipe between two
file descriptors and and puts these in p, where p[0] is for reading and p[1] is
for writing. After that, we create two child processes - one for the left
process and one for the right one. These are the two if-blocks that check if
fork returns 0, which it does in the child processes.

In the left process we close STDOUT. Then we use another system call *dup* that
duplicates a file descriptor. What this means is that we can refer to the same
file or stream but using a different file descriptor. Since STDOUT is closed
and it's the lowest available file descriptor, the write-end of our pipe gets
connected to STDOUT in this process.

Recall that a child process has almost exactly the same memory as a parent
process, and this includes file descriptors, so after we have connected the left
process to STDOUT we want to close the file descriptors in p. If we don't, we
might get subtle bugs, such as getting a *deadlock* where we don't ever see
anything printed. Consider it good hygiene to close a file descriptor when you
are done with it.

After all that, we run the left process and it prints to STDOUT. Similarly, the
right process does almost the exact same thing but for STDIN. And finally, the
parent closes the file descriptors for the pipe and waits for both child
processes to finish (which, if it's a *long-running process*, may never
happen).

## Conclusion

In this article we have used very simple language [^14] to try to understand
what a shell is and how it works. We've looked at how the shell executes a
simple command, how I/O redirection works, and how pipes allow for inter-process
comunication. We've also looked briefly at what files and process are, and how
to use some basic system calls.

I hope I have managed to paint a clear picture of the shell. If you wish to
solidify your understanding of the shell, I recommend you to do what I did and
go through the xv6 shell in detail, and perhaps do some of the suggested
homework in MIT's operating systems class. If you have any comments, please
don't hesitate to [email](mailto:me@oskarth.com) or
[tweet](https://twitter.com/oskarth) me.

[^1]: But conciseness is still more important than completeness.

[^2]: *Abstract away*: To draw away attention from.

[^3]: *Bell Labs*: A special place where a lot of important computer things happened.

[^4]: *Programmers*: People who make computers do what they want them to do.

[^5]: *System calls*: Things you can tell the kernel to do in kernel space.

[^6]: *Directory*: Another word for a folder that contains files and other folders.

[^7]: *Present working directory*: Where you are in the *file hierarchy* at any given time.

[^8]: *Parsing*: Figures out the parts of what you told it are.

[^9]: *C*: The *programming language* most Unix-like systems are written in.

[^10]: *Code*: Text that programmers write that tells computers what to do.

[^11]: *Printing*: Writing it in a place where you can see it.

[^12]: *Buffer*: A place where temporary data is stored.

[^13]: *Block*: Get stuck waiting.

[^14]: Undefined technical terms: xv6, computer program, user interface, hardware, graphical interfaces, user space, kernel space, memory, instruction, data, stack, calling, if-statement, output, input/output, symbolic, binary, format, metadata, pathname, string, characters, devices, integer, argument, status, queue, deadlock, long-running process.
