+++
date = "2015-06-16T15:27:47+02:00"
draft = true
title = "xv6 shell"

+++

This week I started diving into xv6, which is a simple teaching operating-system
based on UNIX b6. There's a nice booklet and the code is on the order of 10 000
lines of codes.


<!--more-->

# Old post
1. Less focus on course.
2. More big on why.
3. More on teach one thing

With this post I'm announcing the OS in 30 day challenge. This is day 0 of my attempt to work through MIT's [6.828: Operating System Engineering](http://pdos.csail.mit.edu/6.828/2014/index.html), extending xv6 and implementing the JOS operating system.

I realized last week how much more productive one is when there's a single
project that's on the [top of your mind](http://www.paulgraham.com/top.html).
Excluding travel days, EuroClojure, coaching ClojureBridge, and visiting family,
I'm canceling almost all of my other commitments until the end of July. This
leaves me with exactly 30 full work days, starting tomorrow. The important thing
though isn't the precise number of days but rather the commitment and deadline.

## What are you gonna do exactly?

I'm gonna do the homework and lab assignments that people at MIT normally do
when they take this course. The first part of the course is about extending xv6
- xv6 is a "simple, Unix-like teaching operating system", that's based on UNIX
v6, but implemented in C. The second part of the course consist of making an OS
called JOS, which is supposed to teach you some modern operating system
concepts.

## Why are you doing this?

When I quit my last job, I asked the CTO, a programmer whose ability I highly
respect, what he thought were my biggest weaknesses as a programmer. His answer
was clear: systems programming and type systems.

On the late Richard Feynman's blackboard you can read: *Know how to solve every
problem that has ever been solved* and *What I can't create I don't understand*.
This is of course just an ambition. And while I don't share his ambition for
being able to solve every problem that has ever been solved, I think he's spot
on when it comes to achieving deep understanding. Consider this my attempt at
grokking systems programming.

C and UNIX have been around for 30 years and they are likely to be around in
another 30 years. While I don't have much experience writing C, I have a lot of
respect for it as an abstraction. Together with Lisp, it strikes me as cleanest
model of programming that exists (I took that one from [Paul
Graham](http://www.paulgraham.com/rootsoflisp.html)).

## Tell me more.

I have no idea what I'm getting myself into. This book is based on a course at
MIT, 6.828. This course has, among other things, a pre-requisite of the material
in course 6.033 which in turn has a pre-requisite 6.004 (which I know probably
less than 50% of). I guess that means: better get to work. Is it feasible? I
guess I'll find out soon enough.

So why have a deadline? A clear goal makes it easier. The worst thing that can
happen is that I fail miserably, and then I can figure out why. I doubt I'll
spend the rest of my software life working on operating systems, but I'd like to
have some understanding of how they work.

For people who are interested in following along, it might helpful to know where
I stand, skill-wise. Here's some things that I have already:

- General programming skills, especially Lisp and web-related things.
- Familiarity with Linux/FreeBSD, including very basic scripting skills.
- Done the first part of the Nand to Tetris book (estimated 50 hours effort).
- Familiarity with C (but really not a whole lot).

## How can I follow along / How can I help?

Every day I'll put out an entry in my code journal that's max 200 words. This is
where I write about what I got done that day. The preferred way to consume this
is with curl: `curl http://plan.oskarth.com/8`, but a normal web browser works
[too](http://plan.oskarth.com/8).

Every week I'll teach some concept in a post that's around 1000 words long. For
these posts I'm going to imagine that a friend of mine is doing the same thing
and is a few days behind, and I'm trying to teach him or her concepts.

If you have some ideas of things you'd like to read about, please let me know
via email or twitter. And if you have knowledge of the material and want to
help, let me know and I'll bombard you with questions!

## Starting tomorrow.

How hard can it be? I have no idea. Let's find out.

I'm excited. Let's get started!
