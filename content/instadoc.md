+++
date = "2015-06-07T18:00:00+02:00"
title = "Instadoc - quick access to documentation"
+++

If you program in multiple languages, you probably don't know every
language's core functions and standard library by heart. Being able to
look up documentation and source code quickly is vital for staying in
the flow. Here's a test: can you look up your language's documentation
in less than 10 seconds?

<!--more-->

In Clojure I've always found
[clojure.repl](https://clojure.github.io/clojure/clojure.repl-api.html)
to be extremely useful in looking up documentation, source code and to
search for functions using `apropos`.

Of course, you can always use online resources to look up
documentation. While useful, they require you to be online, are
sometimes confusing, and it can sometimes take a while to find things;
it's not uncommon that looking something up online breaks my flow.

## Hypothesis

Ten seconds is a somewhat arbitrary limit, but I chose it for a
specific reason: it's widely seen as the limit for
[keeping a user's attention](http://www.nngroup.com/articles/response-times-3-important-limits/). While
that research was done for more passive tasks, I thought it was a good
starting point. For example, if I have to type something really
complicated to get the source code of a function, I'm much less likely
to do it. Here's my hypothesis:

*In most languages you can look up documentation and source code in
less than ten seconds, using built-in, offline tools.*

Additionally, I was interested in looking up examples and finding
code, i.e. search a.k.a. apropos.

When I say ten seconds, the important thing isn't the absolute time,
but rather that it takes a very short period of time. For example,
writing `man strncpy` in a terminal, or evaluating `(apropos "byte")`
in a Clojure REPL gives you results almost instantly (it still takes
some effort to type though, and there are tools in Clojureland that
gives you documentation of a function as you mouse-over it, which
takes less than 0.1 seconds and is thus perceived as happening
instantaneously).

## Results

I took a look at a few languages I have used. There are probably
errors in the following table, and I would love to be corrected. I
found no dedicated tools for code examples, instead they are often put
at the end of the documentation string, if there are any at all.

Here are the preliminary results:

| Language | Shell   | REPL           | Source         | Search         |
|----------|---------|----------------|----------------|----------------|
| Python   | `pydoc` | `help`         | `inspect`      | `pydoc`        |
| Clojure  | No      | `clojure.repl` | `clojure.repl` | `clojure.repl` |
| Scala    | No      | No             | No             | No             |
| Go       | `godoc` | N/A            | No             | No             |
| C        | `man`   | N/A            | No             | No             |

I put up a live spreadsheet that you can see and edit
[here](http://experiments.oskarth.com/instadoc-live/).

## Conclusion

It's probably too early to draw any real conclusions from the limited
data, but I was surprised to find out how hard it seems to be to find
source code in Scala, Go and C, and that Scala's offline documentation
is so bad.

Regardless, I hope this very incomplete list is useful for someone,
and hopefully it can serve as a starting list for a more complete
collection of getting documentation more quickly. I will update this
post as I get more data.

