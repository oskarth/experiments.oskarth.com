+++
date = "2016-03-15T17:06:02+02:00"
title = "Spaced repetition and practice"
+++

How do we become fluent in skills quickly? How can we retain that knowledge with
the minimum amount of effort?

<!--more-->

The last few months I've been doing a bunch of experiments centered around
practice and retention. If you are interested in reading all of them, you can
read them at plan.oskarth.com (entry 18 to 25). In this post I want to focus on
the last experiment I did, and end with some thoughts on what I think it means.


TODO: Mention something about Ebbinghaus?

## Hypothesis

Given that I've *learned* something and added multiple cards to my *spaced
repetition system* (SRS), I'll be able to:

(a) solve the same problem in roughly the same amount of time, regardless of how
long it's been since I did it the last time (time-invariant).

(b) solve problems I've previously learned to solve twice as fast as if I don't
use spaced repetition.

## Methodology

I worked on 10 small problems until I (a) felt like I understood them (b)
solved them in ~5-10m from scratch. After that, I added a bunch flashcards to a
spaced repetition program called Anki. I tried a mix of cards that are all
possible to do in one's head. Examples of types of cards include:

1. Checks for conceptual understanding
2. Basic debugging: what's wrong with this piece of quote
3. Complete a piece of tricky code
4. Listing exhaustive cases to check for some simple domain
5. Basic syntax and idioms questions

See the my notebok [http://plan.oskarth.com/22] for more concrete examples.

Protocol: I've precomputed a series of 5 As and 5 Bs in a random order that I'm
unaware of. I'm doing 10 exercises from Eloquent Javascript that I failed to
produce the "optimal" solution in 5m. For each exercise I make a few flash cards
and add them to my SRS. After doing that, I check the next draw in my random
sequence, and if I get an A I keep the cards, and if I get a B I suspend it
(i.e. I won't see it again).

Doing this, I believe I've eliminated any bias I have in the effort I put into
learning each thing, and to make the cards themselves. I believe the effect will
be more pronounced with time, but I'm going to cap the delay from learning to
testing to an average of two weeks. Each practice test will also be capped to 20
minutes.

## Result

I tested myself on the 10 problems I previously learned (see 24). Recall
that I split the problems into two groups, one where I practiced spaced
repetition (group A) and one where I didn't (group B). On average, it's been
three weeks since I last solved these problems.

Results: Group A took 18 minutes in total, and all problems were solved in
around 5m. Group B took 46m to solve in total. Group B includes one that I
didn't solve (counted as 20m) and one partial solution that took 11m. I decided
to count the partial solution as a solution as it was mostly correct.

I was thus unable to falsify my hypothesis. This has some caveats though. I just
want to mention one here: Anki tracks the total time spent per card, and summing
up all the cards I did the total time spent rehearsing was 19 minutes.

## Conclusion and reflection

Some themes I want to discuss:

- When does it matter if we forget?
- On the value of fluency
- Problems with solution and problems with no clear solution

Can we retain know-how by doing spaced repetition of know-what? I.e. not
actively practicing but reminding ourselves of basic concepts.

Spaced repetition only makes sense if you've learned something. I "knew" this
and had certainly read about it, but I hadn't really internalized it before.

