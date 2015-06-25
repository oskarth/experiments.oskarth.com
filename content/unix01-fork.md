+++
date = "2015-06-20T19:38:21+02:00"
draft = true
title = "Forks and pipes"

+++

When I find a new concept I like to look up the etymology of the word. Here are
some concepts we will look at today.

**shell**: *Like a seashell, an outer layer.*

**file**: *French for a row.*

**fork**: *To divide in branches, go separate ways.*

**pipe**: *An instrument where you blow in one end and music comes out the other.*

## The Shell

## The File

## The Fork


## The Pipe

This post is part of my ongoing experiment in grokking xv6. In it I will teach
you what a shell is, what it can do, and how it does this, using very simple
language.

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
Thompson*, *Dennis Ritchie*, and *Brian Kernighan*.

The shell lives in *user space*, along with most other programs, as opposed to
in *kernel space*. The way it talks to the kernel is by *system calls* (things
you can tell the kernel to do). These system calls allows the user to do things
like open *files* and create *processes*.

## A basic example

If you want to see a list of *directories* (another word for folders) in your
*present working directory* (where you are in the *file hierarchy* at any given
time) you can use the program *ls*. Typing "ls" in a shell and pressing enter
runs it. How does this work? The shell *parses* what you wrote (figures out the
parts of what you told it are) and then it will try to run the program using the
*exec* system call and an *array* (a type of list) of *arguments*.

How does exec work? To understand that we have to take a step back and
understand what a process is. Recall that an operating systems allows several
programs to run on one computer, and processes is a big part in how it does
that. A process in Unix-like systems is a program that runs and has access to
its own piece of *memory* which contains the program's *instructions* (what the
machine is actually executing), *data* and *stack*. The operating system then
makes sure each process gets to do what it wants to do in some reasonable
manner. The system call exec ...

Forgot to mention fork before! When is it called?

Forks and pipes
