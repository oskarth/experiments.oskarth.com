+++
date = "2015-06-20T19:38:21+02:00"
draft = true
title = "What is a shell and how does it work?"

+++

This post is part of my ongoing experiment in grokking xv6. In it I will teach
you what a shell is and how it works, using very simple language.

<!--more-->

If you haven't read first part in this series, you can read it
[here](experiments.oskarth.com/unix00/).

## What is a shell?

A *shell* is just another *computer program*. It is the main *user interface* to
*operating systems* that are similar to *Unix*. An operating system is
responsible for having several programs run on one *computer*, and also to tries
to *abstract away* (to draw away attention from) the specific *hardware* the
computer is running on, so the same program can run on many different types of
computers. Unix is a special type of operating system that was developed at
*Bell Labs* (a special place where a lot of important computer things happened)
by *programmers* (people who make computers do what they want) like *Ken
Thompson*, *Dennis Ritchie*, and *Brian Kernighan*. A *kernel* is the part of
the operating system that we will be concerned with. Modern operating systems
also provide things like *graphical interfaces*.

The shell lives in *user space*, along with most other programs, as opposed to
in *kernel space*. The way it talks to the kernel is by *system calls* (things
you can tell the kernel to do). These system calls allows the user to do things
like open *files* and create *processes*.

## Hello shell

If you want to see a list of what's inside the *directory* (another word for
folder) in your *present working directory* (where you are in the *file
hierarchy* at any given time) you can use the program *ls*. Typing `ls` in a
shell and pressing enter runs it and returns the contents of that folder. How
does this work? When you press enter the shell *forks* to a *child process*,
which *parses* what you wrote (figures out the parts of what you told it are)
and then runs that parsed *command*.

Let's take a step back. What's a fork? And what exactly is a process? Recall
that an operating systems allows several programs to run on one computer, and
processes is a big part in how it does that. A process in Unix-like systems is a
program that runs and has access to its own piece of *memory* which contains the
program's *instructions* (what the machine is actually executing), *data* and
*stack*. The operating system then makes sure each process gets to do what it
wants to do in some reasonable manner. Fork is a system call that allows a
process to create another process. The process *calling* (another word for
saying "do this" code) fork is called the parent process, and the process it
creates is called the child process. The child process has almost exactly the
same memory that its parent process has.

In *C* (the *programming language* most Unix-like operating systems are written
in) *code* (text that tells computers what to do) it would look something like
this:

    int pid = fork();
    if(pid == 0)
      runcmd(parsecmd(buf));
    wait();

When we call fork we get back a *pid*, or a process id. We now have two
processes, and pid is different in the two. In the parent process, the pid is
some number that is used to identify the process, whereas in the child process
it's simply equal to 0. Thus the runcmd function will just run for the child
process. The parent process instead will keep executing and get stuck at *wait*,
which is another system call that waits for a child process to return. Once the
child process has finished running, the parent process - which is the shell -
will resume running, and we can type another command.

## Redirection

Let's say we want to save the *output* from running `ls` above. We can do so
using something called *I/O* (input/output) *redirection*:

    ls > foo

There are three parts to this command. When the shell parses the command, it
figures out that it's a redirection from `ls` to `foo`. After the shell has
forked to a child process it runs `ls` and saves the output in a *file* called
foo. A file is a collection of *characters* that is somewhere in the computer,
written in some *format*, and with some *metadata* associated with it (such as
who is allowed to read and write to it), and we can access it by using a
*filename* as a *unique identifer*. In the above example the full filename for
foo for me is `/Users/oskarth/foo`. In general, the more UNIX-like a system is
the more things are abstracted away as files.

How does the output of ls end up in foo? To understand that we have to know a
bit more about files work in Unix. The system call *open* is used to see the
contents of a file. When we open a file to read or write to it, we get a *file
descriptor* back. This file descriptor is just an *integer*. There are three
special integers, 0, 1, and 2. These are called, in order, *STDIN*, *STDOUT*,
and *STDERR*. Another word for these is *standard streams*. Streams and files
are closely related - the difference is that streams don't necessarily end,
since a process can keep writing to it while another process is reading from
it.

When we run `ls`, it returns the results by *printing* it (writing it in a
place where you can see it) to STDOUT, and STDOUT is what we see in the shell.
File descriptors are handed out by the kernel starting from the lowest
available file descriptor, and when we start the shell all standard streams are
already open.  We can use this fact to get I/O redirection with something like
this:

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
status of processes. If we want to have a list of all my processes sorted by
their process id we can *pipe* the result of running ps to the program *sort*.

    ps | sort


When the shell parses this command it sees the symbol "|" and knows it's a pipe
command. A pipe command has two sides: a left and a right side. When we write
on the left side we can read from right side. A pipe is a small *buffer* (a
temporary place) that lives in kernel space and allows processes to talk to
each other, which is called *inter-process communication*. This is different
from redirection since it happens continuously, as opposed to writing to some
temporary file, and then once that's done reading from it. It's also a *queue*,
so even if new data comes in faster than you can process it in the right
process the data doesn't disappear, and it doesn't *block* (get stuck waiting)
either.

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

We create an array of two integers, which is where we keep track of our file
descriptors. We then use the system call pipe, which creates a pipe where p[0]
is for reading and p[1] is for writing. After that, we create two child
processes - one for the left process and one for the right one. These are the
two if-blocks that check if fork returns 0, which it does in the child
processes.

In the left process we close STDOUT. Then we use another system call *dup* that
duplicates a file descriptor. What this means is that we can refer to the same
file or stream but using a different file descriptor. Since STDOUT is closed
and it's the lowest available file descriptor, the write-end of our pipe gets
connected to STDOUT in this process.

Recall that a child process has the same memory as a parent process, and this
includes file descriptors, so after we have connected the left process to
STDOUT we want to close the file descriptors in p. If we don't, we might get
subtle bugs, such as getting a *deadlock* where we don't ever see anything
printed. Consider it good hygiene to close a file descriptor when you are done
with it.

After all that, we run the left process and it prints to STDOUT. Similarly, the
right process does almost the exact same thing but for STDIN. And finally, the
parent closes the file descriptors for the pipe and waits for both child
processes to finish (which, if it's a *long-running process*, may never
happen).

## Conclusion

In this article we have used very simple language to try to understand what a
shell is and how it works. We've looked at how the shell executes a simple
command, how I/O redirection works, and how pipes allow for inter-process
comunication. We've also looked briefly at what files and process are, and how
to use some basic system calls.

I hope I have managed to paint a clear picture of the shell. If you wish to
solidify your understanding of the shell, I recommend you to do what I did and
go through the xv6 shell in detail, and perhaps do some of the suggested
homework in MIT's operating systems class. If you have any comments, please
don't hesitate to [email](mailto:me@oskarth.com) or
[tweet](https://twitter.com/oskarth) me.
