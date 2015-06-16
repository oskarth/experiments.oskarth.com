+++
date = "2015-06-14T23:00:00+02:00"
title = "Writing A Lisp Interpreter"

+++

I've always really liked Lisp as a programming language, but I've yet to write
an interpreter for the language. This week I decided to change that.

<!--more-->

**Hypothesis:** I can write a lisp interpreter in under 500 LOCs and in under a
week.

Since the beginning of last week I've been keeping a [code journal](https://twitter.com/oskarth/status/608397165925437441). To give you a
feel for process so far of this project, I'm posting some excerpts from it
related to this project.

**Tuesday**

> I'm working on my own little Lisp interpreter, based on the one described in SICP. My goal is to get it to a reasonable state by the end of this week, and then we'll see.

**Wednesday**

> I got stuck in a code mess where I had a bad mental model of what was happening when I was encoding and calling built-in primitive functions. I decided to start over using LISP 1.5 Programmer's Manual and a legal pad.

**Friday**

> Earlier I had started with the eval and apply functions, and then tried to add the environment and all the other "details", such as primitive procedures and their bindings, ad hoc. This meant that I often didn't have working code, and my mental model suffered as a result.

> Today I started from the bottom-up, which is generally a much better way to do Lisp programming, so my confidence in the code and its workings grew. When I found bugs, I knew almost straight away where the problem was.

> I can now evaluate basic lambdas with proper scoping. The biggest problem was to reconcile the lack of mutable cons cells in Racket with the design SICP has for its environment model. I wasted a lot of time using mutable-cons, which infected the rest of my program. Eventually I decided to remove definitions and assignments, until I figured out how to deal with this mismatch. I also haven't implemented conditionals yet.

**Saturday**

> I saw someone on Github who solved the issue with mutable pairs. Essentially it was the same as what I had done, with the addition of a function that converts all user input lists to mutable lists, along with a convenience macro that allows us to use mcadr, and other nested access functions. This strikes me as a hack that works, but not as the right solution.

> After talking to some helpful people on IRC, especially technomancy of Leiningen and Atreus fame, I got some pointers on how to approach the problem. I decided to keep the local environment immutable, and just create a hash-map for the mutable top-level, for defining functions and such.

> I also gave the lisp interpreter a real name: Sai. It's my little celebration of to the spirit in the machine. I'd like to keep working on it more - for example, I'd like to implement a macro system and play around with that. We'll see if or
when time permits. 

Here are two basic program that demonstrates functions, definitions, conditionals,
scoping, and basic data types.

```
(((lambda (x) (lambda (y) (+ x y))) 3) 4) ; => 7

(define (append x y)
  (if (null? x)
    y
    (cons (car x) (append (cdr x) y))))

(append '(a b c) '(d e f)) ; => '(a b c d e f)
```

Sai is on Github [here](https://github.com/oskarth/sai).

## Conclusion and future work

Yes, it was possible, and it ended up being less than 200 lines of code. It's
very rudimentary and don't support a lot, so that number doesn't mean that much.

Of course, since I was mostly following the path laid out in SICP it's not that
surprising. But I do know a lot more now than I knew a week ago, and I've a
feeling this project helped with my intution for how programming languages work.

There's a few different things I'd like to play around with: implementing a
macro system, Clojure-like data structures, minimizing the amount of primitives
in the eval function, write some real programs in it, write the interpreter in a
different language, etc.

There's a lot to do and I've barely scratched the surface of programming
language design, but so far it's been very fun and rewarding, and I recommend you
to implement your own lisp interpreter!

*P.S. I might change the format of this experimental journal. For example, I
might change the posting frequency to every other week, or write about one
experiment for several weeks in a row. If you have any thoughts on what kind of
format you think would be interesting to read, please let me know. D.S.*
