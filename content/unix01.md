+++
date = "2015-06-20T19:38:21+02:00"
draft = true
title = "shell"

+++

## What is a shell?

A *shell* is just another *computer program*. It is the main *user interface* to
*operating systems* that are similar to *Unix*. An operating system is
responsible for having several programs run on one *computer*, and also to tries
to *abstract away* (to draw away attention from) the specific *hardware* the
computer is running on, so the same program can run on many different types of
computers. Unix is a special type of operating system that was developed at
*Bell Labs* (a special place where a lot of important computer things happened)
by *programmers* (people who make computers do what they want) like *Ken
Thompson*, *Dennis Ritchie*, and *Brian Kernighan*.

The shell lives in *user space*, along with most other programs, as opposed to
in *kernel space*. The way it talks to the kernel is by *system calls* (things
you can tell the kernel to do).  These system calls allows the user to do things
like open *files* and create *processes*.

A *system call* is something you can tell the kernel to do.
