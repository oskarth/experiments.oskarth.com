+++
date = "2015-06-20T16:00:25+02:00"
title = "Grokking xv6"

+++

This is my first post in a series on grokking xv6, a simple Unix-like teaching
operating-system. *To grok something* means to understand it intuitively or
empathetically.

<!--more-->

xv6 is a modern rewrite of Unix V6, the first Unix that was published outside of
Bell Labs, and John Lion's commentary of its source code. It consist of a text
and the source code. In total the source is under 10 000 lines of code, and the
book is under 100 pages. You can find out more at MIT's Operating Systems
Engineering [class website](http://pdos.csail.mit.edu/6.828/2014/xv6.html),
which is where xv6 was written.

## Motivation

When I quit my last job, I asked the CTO, a programmer whose ability I highly
respect, what he thought were my biggest weaknesses as a programmer. His answer
was clear: systems programming and type systems. What is systems programming?
The short answer is: software that isn't application software.

On the late Richard Feynman's
[blackboard](http://caltech.discoverygarden.ca/islandora/object/ct1%3A483/datastream/JPG/view)
you can read: *Know how to solve every problem that has ever been solved* and
*What I cannot create I do not understand*.  This is of course just an ambition.
And while I don't share his ambition for being able to solve every problem that
has ever been solved, I think he's spot on when it comes to achieving deep
understanding. Consider this my attempt at grokking systems programming.

C and UNIX have been around for 30 years and they are likely to be around in
another 30 years. While I don't have much experience writing C, I have a lot of
respect for it as an abstraction. Together with Lisp, it strikes me as cleanest
model of programming that exists (I took that one from [Paul
Graham](http://www.paulgraham.com/rootsoflisp.html)).

Regardless of what you end up doing as a programmer, it's extremely helpful to
have a grasp of the fundamentals. UNIX and its philosophy have such a huge
influence on programming at large, there's nothing you can do that they don't
touch. Even if you are making a web app, once it reaches a certain size you are
bound to run into issues that have already been studied at length in the world
of operating systems.

## Hypotheses

**1. With dedicated study, I'll be able to grok xv6 in about a month.**

To make the concept of grokking more concrete:

- understand every single line in the source code
- teach it or parts of it to someone else
- rewrite or extend parts of the source code

The first will be more of a qualitative test to apply to myself, I aim to do the
second in these posts, and the third will be done with the help of MIT's 6.828
[homework](http://pdos.csail.mit.edu/6.828/2014/schedule.html).

**2. After grokking xv6 I'll have a firm grasp of fundamental OS concepts.**

This can be tested by:

- ability to understand modern operating systems
- contributing a patch to a modern OS like Linux or FreeBSD

See this HN [comment](https://news.ycombinator.com/item?id=4599048) for why the
second wouldn't be as unreasonable of a goal as it may appear at first.

**3. After grokking xv6 I'll have developed working knowledge of C.**

This can be tested by understanding and / or contributing to a non-trivial open
source project written in C.

## Methodology

In future posts I will choose something specific to teach and focus each post on
that. In that sense it can best be seen as one long post, with this being the
introduction. After having finished with xv6, I'll write some sort of
conclusion. In other words, this experiment is a work in progress.

If you wish to follow along, you can do so on a weekly basis here. You can also
read my daily code journal. The latest entry can be found by typing `curl -L
plan.oskarth.com` into your terminal. If you want to see previous entries you
can find them at `curl plan.oskarth.com/list`. All the code is available on
[Github](https://github.com/oskarth/xv6).

(If you liked this, you might enjoy
[What is a shell and how does it work?](http://experiments.oskarth.com/unix01/). To
stay up to date on my experiments, consider
[subscribing](http://eepurl.com/bvtdfj).)
