+++
date = "2015-05-08T16:00:00+02:00"
title = "Law Of The Instrument"

+++

It's tempting to solve the problem you already know how to solve, as
opposed to the problem that matters. This is true even if you are
aware of it.

<!--more-->

This March, I introduced Unfolds in a
[blog post](http://blog.oskarth.com/unfolds-a-jungle-of-ideas-prototype). Seen
as an experiment, Unfolds was a failure. It's on the order of 500
lines of Clojure/Clojurescript code, and despite being my main hacking
project for about a month, it failed to test an actual hypothesis.

What I set out to do with Unfolds was to get to the gist of an idea in
a few hundred words. This is mostly a problem of writing these gists
clearly and concisely. Without that you have nothing. I was well aware
of this, but yet I approached the problem by spending my time writing
a tool for creating and browsing these gists. What went wrong? We can
get a clue by listening to the words of Abraham Kaplan:

> I call it the law of the instrument, and it may be formulated as
> follows: Give a small boy a hammer, and he will find that everything
> he encounters needs pounding.

My hammer was programming, and I was pounding away at a tool that
served a subordinate purpose to gists that don't even exist yet.

The real problem here is so difficult, and I hadn't sufficiently
deconstructured and simplified it, that I ended up trying to solve a
different problem. It doesn't matter if what you are building is
clever if it doesn't solve a real problem. An hypothesis that can
reasonably be falsified would function as a compass, and keep the
pursuit honest.

Would making a tool necessarily be a bad idea? No, but that would be a
different direction and a different hypothesis. If you are building a
house with just a hammer, you'll have a bad time cleaning windows.

## What would a real hypothesis look like?

The idea behind Unfolds is still potent, and there are many questions
and hypotheses hiding in it. Here are a few sketches of assertions
that can be tested.

1. You can communicate the gist of an idea in less than 200 words. By
   gist we mean that reading these words will be enough for most
   research purposes.

2. The first 200 words of a Wikipedia article do not satisfy the
   metric in 1.

3. This author can explain ten concepts in under 200 words. This is
   only true for concepts that are familiar to him.

4. Images and illustrations are vital in a few select domains, but not
   needed in the majority of explanations. Specifically, there's a
   trade-off in time investment, and it's usually not worth it.

5. There exist heuristics which make a short explanation particularly
   good or particularly bad. These can be discovered. It's possible to
   build tools that encourage good explanations.

I'm sure there are many more, but these are things on the top of my
head that'd be interesting to investigate. Some assertions are easier
to test than others, and I'll probably revisit the matter soon.
