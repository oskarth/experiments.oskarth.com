+++
date = "2015-05-22T22:00:00+02:00"
title = "Nand to Tetris 1, with Dan Luu"

+++

In the last few weeks I've been working my way through the excellent
book *Elements of Computing Systems - building a modern computer from
first principles* as part of the equally excellent Nand to Tetris
MOOC.

<!--more-->

I started reading the book a few years ago when I attended Recurse
Center, but instead of completing it I ended up
[writing a domain specific language](http://blog.oskarth.com/writing-a-dsl-in-clojure)
for the first few chapters. A useful exercise, no doubt, but this time
around I intend to finish the whole book.

The main reason I want to go through the book is because I want to
have a better sense of how a computer works, and get a rough idea how
one could build one. In a sense its value is proportional to how well
it serves as a mental model for the real world.

So how does it stack up? This is the first of a multi part series,
starting with the first three chapters of the Elements of Computer
Systems book, on boolean logic, boolean arithmetic, and sequential
logic. I wrote down a collection of assertions that I wanted to diff
with what's out there in the real world. [Dan Luu](http://danluu.com/)
was gracious enough to give me some great answers based on his
expertise in the field. His answers are in italics.

While I highly recommended that you've taken the equivalent of a Nand
to Tetris course, this is not strictly necessary. With some luck this
series will convince you to embark on a similar project on your own.

Let's begin.

**1. Any boolean function can and usually is built from NAND gates.**

*While this is conceptually accurate, this usually isn’t done in
 practice. There are multiple reasons for this, but it mostly comes
 down to cost, power, and performance. The performance aspect is that
 you can directly implement functions with transistors more
 efficiently than you can with NAND gates. You can, very loosely,
 think of this as similar to how compilers sometimes inline functions
 and then do optimizations across inlined functions.*

This property, that any boolean function (i.e. any truth table) can be
built using just NAND gates, is called functional completeness, and
the proof is quite neat. Consider a truth table for some function and
some variables. Each row where the function evaluates to true can be
represented by ANDing together the variables, which are represented
either as true or NOT true. We then OR together all rows to get a
complete representation of that function’s truth table. For example,
Xor(a,b) evaluates to true when either a or b is true. We can
represent this as follows: OR(AND(a, NOT(b)) , AND(NOT(a), b)). We can
thus express any boolean function using just AND, NOT and OR. It then
turns out, using
[De Morgan's laws](https://en.wikipedia.org/wiki/De_Morgan's_laws) and
similar logical relationships, that we can express AND, NOT, OR in
terms of NAND.

Another cool thing Dan taught me is why NAND gates are usually
prefered over NOR gates, despite both of them being functionally
complete. If you are interested in that, you can read more
[here](https://electronics.stackexchange.com/questions/110649/why-is-nand-gate-preferred-over-nor-gate-in-industry). However,
we did manage to get to the moon in the 60s using just
[NOR gates](https://en.wikipedia.org/wiki/Apollo_Guidance_Computer#Design).

**2. Logical functions are built up from more elementary ones.**

*Again, this is conceptually correct, but for performance reasons
 people sometimes build logical functions directly from
 transistors. For much more detail on this,
 [Weste & Harris](http://www.amazon.com/CMOS-VLSI-Design-Circuits-Perspective/dp/0321547748)
 is great. For a quick explanation, see
 [this](http://www.cerc.utexas.edu/~jaa/vlsi/lectures/3-1.pdf). That
 explanation isn't self contained. Some things you want to know are
 that the funny symbol near the bottom of those diagrams is ground
 (0), the funny line/symbol at the top is the on voltage (1). Then you
 have the transistors. If there's a bubble on the gate (input), that's
 a PMOS transistor. It turns on (conducts) when the input is 0, and
 it's good at passing "1"s. Otherwise, it's an NMOS, and has the
 opposite properties.*

Similar to the answer above, the interface is correct but the implementation is naive to the point of being misleading.

**3. Integers are represented by two's complement.**

*Yes, 1's complement is rarely used, although there are some applications where it’s superior. You might also be interested in logarithmic and residue number systems, which make some operations easier (faster) at the cost of making other operations slower. For more on that, [Koren](http://www.amazon.com/Computer-Arithmetic-Algorithms-Second-Edition/dp/1568811608) has a really nice text.*

*Also, in contrast to the address you build in nand2tetris, adders are
commonly built using some kind of parallel prefix tree to reduce the
delay (i.e., increase the
performance). [Carry-lookahead adders](http://en.wikipedia.org/wiki/Carry-lookahead_adder)
are probably the simplest form of this, but they’re not usually the
fastest thing you can do. The Weste&Harris book mentioned above has a
lot more information on different types of prefix trees.*

This one was funny, as I think of a carry-look-ahead as a neat
optimization, whereas in the real world it’s too slow to use by
itself.

**4. When adding integers in a real-world Adder, overflows are ignored.**

*It depends! It’s not an error, but there’s often an output from the
 ALU that signals an overflow.*

**5. Our ALU is essentially the same as a real one.**

(Our ALU has two 16-bit inputs, six control bits, two output flags,
and one 16-bit output).

*It’s missing
 [pipelining](http://en.wikipedia.org/wiki/Pipeline_(computing\)),
 [forwarding](http://en.wikipedia.org/wiki/Operand_forwarding), and
 other performance optimizations, but, fundamentally, it does the same
 stuff as a “real” ALU. Real is in quotes since it’s no less real than
 any other ALU, although “real” ALUs usually implement many more
 functions, have more control bits, etc. :-). Also, some “real” ALU
 operations can also take several clock cycles, unlike the one in your
 design.*

**6. There's one master clock that keeps track of computer time.**

*This is correct for designs that are made to be simple. However, in
 “real” designs there is often more than one clock for a multitude of
 reasons, such as dealing with I/O devices that run at different
 speeds. For more on how to deal with that, see
 [this](http://www.sunburst-design.com/papers/CummingsSNUG2008Boston_CDC.pdf).*

I would say this makes my mental model incorrect, in that I would
expect one clock cycle to be the unit that everything in hardware
uses, but at second thought I see why that wouldn’t make sense with
I/O-bound hardware parts.

**7. Our Register, RAM and Program Counter are essentially realistic.**

(In addition to 16-bit input and outputs, we have the following: Our
Register has a load bit, our RAM a load bit and an n-bit address, and
our Program Counter has a load, inc, and reset bit).

*This is true conceptually/logically. However, in “real” systems
 register files are often custom circuits built at the transistor
 level, and RAMs are also custom. On chip RAMs are usually SRAMs,
 which are built out of transistors like other chip logic (although
 they typically have an analog component to them, unlike the logic
 you’ve built). Off chip DRAM is a totally different beast. There’s
 also normally multiple read/write ports, as opposed to just one
 combined read/write address.*

This also makes sense, but the shared memory bit sounds scary. Yet
another rabbit hole to go into, for a rainy day.

## Conclusion

In general, most of my assertions were right on an interface level,
but wrong on an implementation level. Is this a good mental model? I
think so. Unless you are building a real computer it’s good enough,
conceptually. You could also, theoretically at least, build a computer
using the tools given to you in Nand to Tetris that would be similar
to an Intel machine from the early 80s, which isn’t that bad.

In the next part we’ll move higher up the stack, looking at machine
language, computer architecture and an assembler.
