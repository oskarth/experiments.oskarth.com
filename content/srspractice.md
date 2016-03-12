+++
date = "2016-03-15T17:06:02+02:00"
draft = true
title = "SRS Practice"

+++

Start by recapping things I've done. This is essentially my lab book.

## Excerpts from my lab book

16:

```
Many small experiments


Today is January 10th, 2016. This year I want to do many small experiments, and
do write-ups of a few of them on experiments.oskarth.com. This will thus be a
kind of lab notebook, rather than only being about code per se.

Many experiments have been started but have not reached a conclusion
(github.com/oskarth/{instantetymonline, praxis, attentions}). I have noticed
that the longer I've worked on something, the longer the estimated time of
completion is. I.e. spending 1 day on something, it's likely to take 1 more day,
and if I spent a month on something it is likely to take another month to
finish. Others (http://www.johndcook.com/blog/2015/12/21/power-law-projects/)
have noticed this as well.

I think a useful attitude to guard against this is to be paranoid about things
taking longer than a few days, especially if it isn't your main commitment in
life.

As a first experiment for this new year, I'm looking into deliberately
practicing SQL. Hypothesis: After working through all the exercises at
pgexercises.com once, the second time will take 1/3 the time, and the third time
a 1/3 of the second time.

* 30 minutes of SQL deliberate practice at pgexercises.com
```

17
```
Re-scoped SQL practice and some meta-lessons


Today is January 12th, 2016. Considering what I said last time about being
paranoid about things taking more than a few days, I decided to limit the scope
to just the two first sections of pgexercises.com. Here's my progress so far:

Run 1: 8th to 11th January, 30-45m sessions. Total time: 2h 15m.
Run 2: 12th January, 55m.
Run 3: Hopefully tomorrow.

I conducted the tests as a sort of "open book exam". I didn't read "Learn SQL"
or anything like that beforehand, but instead looked things up as I got stuck.

I'll clock in just under a week if I finish tomorrow, and if I hadn't limited
the scope severely I might never have finished. Some meta-lesson are already
clear:

1. Ballpark estimate time for each exercise and take that number times two
2. 30 minutes a day is very little for this type of experiment
3. Being ruthless about cutting scope is good for finishing

* Run 2 of SQL practice, ~1h.
```

18

```
SQL experiment conclusions


Two days ago I finished the third run of my SQL experiment. The general question
I wanted to explore was: how do we become fluent in skills quicky?

As a reminder, my hypothesis was: "After working through all the exercises at
pgexercises.com once, the second time will take 1/3 the time, and the third time
a 1/3 of the second time." Result:

Run 1: 8-11th Jan, 145m
Run 2: 12 Jan, 55m
Run 3: 14 Jan, 35m

My hypothesis was falsified. While the second time around took about a third of the
time, the third run wasn't even twice as fast as the second run.

Observations and notes:
- The 'Basic' section in isolation took ~5m the third run
- I did some SQL outside of these sessions
- The closeness of the session indicates elements of shallow memorization,
  rather than internalization
- I got stuck on certain exercises a lot, such as mistakenly thinking of 
  sub-queries as being namespaced

Questions that were spawned:
- Do you need to learn more complex things in order to master fundamentals?
- How could I have better tackled the exercises/cocnepts that I got stuck on?

* finish SQL experiment
```

19

```
SQL experiment #2 - exploring the practice/theory/practice loop


After my last experiment (see /18) I was reminded on Twitter of 'Effective
learning: Twenty rules of formulating knowledge' and specifically about learning
before memorizing. This lead me to wanting to do the following experiment.

Hypothesis:
After spending one hour working through the Aggregate section at pgexercises.com
once, then reading 'Learning SQL' with the goal of understanding
grouping/aggregates for 1h, I will be able to solve the same problems in 20
minutes a few days later.

Result:
Jan 16: Solved 11 first problems of Aggregates in 1h.
Jan 17: Read Aggregates chapter and parts of Subquery chapter for 1h.
Jan 23: Skimmed some of my notes for 2-3m. Solved same problems in 20m.

Conclusion:
I was unable to falsify my hypothesis. The time between 2 and 3 was longer than
I wanted it to be. I added some inference in skimming the notes beforehand.

Misc notes and questions:
- The problem I had before with subqueries not being 'namespaced' is called
  correlated subqueries, and they are run once for each row.
- Can we retain know-how by doing spaced repetition of know-what? I.e. not
  actively practicing but reminding ourselves of basic concepts.
```

20

```
A new hypothesis for practice and retaining knowledge


After my last two experiments I realized spaced repetition only makes sense if
you've learned something. I "knew" this and had certainly read about it, but I
hadn't really internalized it before.

We often learn things and then forget them. One solution is over-learning, which
happens naturally for things we do a lot. I suspect this is one of the reasons
fundamentals are often learned better once we do more advanced things. For
example, if you grow up speaking German pronouns and cases will be ingrained,
because you've dealt with sentences that depended on them for so long.

What about things that are not naturally over-learned, for whatever reason? Can
we learn things once and easily maintain that skill? 

Hypothesis: Given that I've learned something and added multiple cards to my
spaced repetition system (SRS), I'll be able to solve the same problem in
roughly the same amount of time, regardless of how long it's been since I did it
the last time (time-invariant).

This is quite a mouthful and I'll explain what I mean by 'learning', 'cards' and
SRS with more concrete examples in the days to come.
```

21

```
Preliminary conclusions from practice+SRS experiment


About a week ago I worked on three small problems until I (a) felt like I
understood them (b) solved them in ~5-10m from scratch. After that, I added a
bunch flashcards to a spaced repetition program called Anki. I tried a mix of
cards that are all possible to do in one's head. Examples of types of cards
include:

1. Checks for conceptual understanding
2. Basic debugging: what's wrong with this piece of quote
3. Complete a piece of tricky code
4. Listing exhaustive cases to check for some simple domain
5. Basic syntax and idioms questions

I wrote a RPN calculator in Scheme, a basic aggregation query in SQL, and a word
counter in C. Today I re-did them and was able to solve them all in ~5m. I've
thus not been able to falsify my hypothesis.

What does this mean? I don't know yet, and I'm still skeptical of the results.
Perhaps I simply had these specific solutions fresh in my mind? Where does the
line been memorization and internalization lie?

A thought: Learning is a form of semantic compression of concepts that are
robust under perturbation.

To be continued.
```

22

```
Examples of Anki cards


1. Conceptual understanding
[sql] Whats difference between WHERE and HAVING?
---
WHERE is used to filter what data gets input to aggregate function, HAVING is used to _filter_ output.

2. Basic debugging
[c] What's wrong in the following part of a word-count program?
state = OUT;
while ((c = getchar()) != EOF) {
nc++;
if (c == '\n')
    nl++;
if (c == ' ' || c == '\t' || c == '\n')
    state = OUT;
else {
    state = IN;
    nw++;
}
---
Last clause should be `else if (state == OUT)`, otherwise we'll keep adding new words

3. Complete code
[scheme] Complete the following for an RPN calculator:
(let rpn ((token (next)) (stk '()))
  (cond ((eq? token 'nl) <stuff>)
  ((number? token) <stuff)
  (else ...))))
---
(rpn (next) (cons ((op token) (cadr stk) (car stk)) (cddr stk)))

4. Cases
[scheme] What are the main cases for a "next" fn for an RPN calculator?
---
1. EOF (exit)
2. Space (skip to next thing)
3. Newline (signal done)
4. Else (for datums like numbers and operators)

5. Idioms
[c] How to read character-by-character from STDIN until EOF?
---
while ((c = getchar()) != EOF) { ... }
```

23

```
Alternative explanations to the previous practice+SRS experiment


This is a two week overdue post. Recall entry 21 where I said I was skeptical of
my previous results. Here are some alternative explanations.

Alt 1. Interval too short, problems still fresh in my mind
Alt 2. Problems too easy
Alt 3. Interference in learning
Alt 4. Cards leads to shallow memorization
Alt 5. Spaced repetition small effect, effort of learning and making cards enough

For now, 4 is less of a worry if it leads to being able to do a given tasks in
5m at any future point in time. I also have a hunch that this might be less of a
problem than one might think. 2 is also less of a worry as I'd naturally not
select "too easy cards", since I evidently don't know them well enough, even if
the gotcha is something as simple as syntax or language conventions.

That leaves 1, 3, and 5, which are more serious problems of the system in
question. The next post will show the protocol I'm using to tackle them.

Meta-mistake: I have not been sufficiently paranoid about things taking longer
than a few days (see entry 16).
```

24

```
Practice+SRS experiment, version 2


Hypothesis: Using spaced repetition will allow me to solve problems I've
previously learned to solve twice as fast as if I don't use spaced repetition.

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

Meta-mistake: I started doing the above with some Go stuff which aren't relevant
to me right now, motivation-wise.
```

25

```
Practice+SRS experiment v2, results


Today I tested myself on the 10 problems I previously learned (see 24). Recall
that I split the problems into two groups, one where I practiced spaced
repetition (group A) and one where I didn't (group B). On average, it's been
three weeks since I last solved these problems.

My hypothesis was: Using spaced repetition will allow me to solve problems I've
previously learned to solve twice as fast as if I don't use spaced repetition.

Results: Group A took 18 minutes in total, and all problems were solved in
around 5m. Group B took 46m to solve in total. Group B includes one that I
didn't solve (counted as 20m) and one partial solution that took 11m. I decided
to count the partial solution as a solution as it was mostly correct.

I was thus unable to falsify my hypothesis. This has some caveats though. I just
want to mention one here: Anki tracks the total time spent per card, and summing
up all the cards I did the total time spent rehearsing was 19 minutes.

A longer piece with more fleshed out thoughts will follow soon.
```
