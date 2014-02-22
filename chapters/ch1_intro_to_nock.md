# Introductory Nock

> \noindent *What one fool can do, another can.*
> \medskip \\
> \noindent ---**Ancient Simian proverb**

Think of Nock as a kind of functional assembly language.  It's not like
assembly language in that it's directly executed by the hardware.  It is like
assembly language in that (a) everything in Urbit executes as Nock; (b) you
wouldn't want to program directly in Nock; and (c) learning to program directly
in Nock is a great way to start understanding Urbit from the ground up.

Just as Unix runs C programs by compiling them to assembler, Urbit runs Hoon
programs by compiling them to Nock.  You could try to learn Hoon without
learning Nock.  But just as C is a thin wrapper over the physical CPU, Hoon is
a thin wrapper over the Nock virtual machine.  It's a tall stack made of thin
layers, which is much easier to learn a layer at a time.

And unlike most fundamental theories of computing, there's really nothing smart
or interesting about Nock.  Of course, in a strictly formal sense, all of
computing is math.  But that doesn't mean it needs to feel like math.  Nock is
a simple mechanical device and it's meant to feel that way.

Let's get start by learning how to use Urbit's operating system, Arvo, to
evaluate Nock code.

## Getting Started
\label{sec:getting_started}

We'll assume that you've gone through the Urbit setup process and have an Arvo
prompt that looks something like this:

```text
~tomsyt-balsen/try=>
```

At your prompt, type the following exactly:
```text
~tomsyt-balsen/try=> .*(42 [0 1])
```
This should return `42`. Don't worry about what this is doing yet.

It's very important that you actually go to your prompt and type in our
examples. Copying and pasting is cheating, although using your up-arrow is not. This might seem silly, but to learn
Nock (or any language)  it's very important that your fingers get
comfortable writing it. 

If you accidentally make a mistake typing in a Nock expression, you'll get a
syntax error:

```text
~tomsyt-balsen/try=> .*(42[0 1])
~ <syntax error at [1 6]>

~tomsyt-balsen/try=> .*( 42 [0 1])
~ <syntax error at [1 4]>
```

The easiest way to get a syntax error is to accidentally leave out a space or add
an extra one. Fortunately, the error message tells you where the mistake is; for example, the output

```text
~ <syntax error at [1 4]>
```
means that there's an error at `line 1`, `column 4`.  Once you know where an
error is, it's much easier to fix.

But even if your Nock expression is formatted correctly, you might get
something that looks like this:

```text
~tomsyt-balsen/try=> .*(42 [5 0 1])
! exit
```
This means that the expression you typed in is correct Nock, but it just
doesn't produce anything. Unfortunately, we can't give you line and column
numbers on this one, so the only surefire way to debug an exit message is to
understand what your code is doing. Literally speaking, an exit message means
you tried to do something that just doesn't make sense, such as trying to
reference data that doesn't exist, or trying to increment something that's not
a number, or asking if `42` is equal without asking what it's equal to.

Enough about errors, let's practice some expressions that work:

```text
~tomsyt-balsen/try=> .*(41 [0 1])
41

~tomsyt-balsen/try=> .*(40 [0 1])
40

~tomsyt-balsen/try=> .*(374 [0 1])
374
```

The perceptive reader will notice the pattern here: If a is a number, `.*(a [0 1])` always produces `a`. To test it, run the following, but
replace `a` with any number you like. 

```text
.*(a [0 1])
a
```

Once you're satisfied that this is true, let's do something slightly
different:

```text
~tomsyt-balsen/try=> .*(374 [1 0])
0

~tomsyt-balsen/try=> .*(40 [1 0])
0

~tomsyt-balsen/try=> .*(41 [1 0])
0
```

This pattern is pretty easy: `*(a [1 0])` always produces `0`, no matter what a
is.

Again, play around with the above yourself by choosing your own values for a: 

```text
.*(a [1 0])
0
```

One more pattern, and then we'll actually explain what these numbers and brackets represent:

```text
~tomsyt-balsen/try=> .*(374 [1 374])
374

~tomsyt-balsen/try=> .*(40 [1 374])
374

~tomsyt-balsen/try=> .*(374 [1 40])
40
```

As an exercise, run the last three lines again but replace `374` and `40` with
numbers of your own.

You've probably already guessed the pattern here: `.*(a [1 b])` always produces
`b`, regardless of `a`. But feel free to test it, replacing `a` and `b` with any number. 

```text
.*(a [1 b])
b
```

Let's run the following piece of code again:

```text
~tomsyt-balsen/try=> .*(42 [0 1])
```

Nock is made up of two basic building blocks: atoms, which can be any
non-negative whole number, and cells, which are pairs of numbers or cells
(cells can go inside of other cells.) `42`, for example, is an atom. `[0 1]` is
a cell. Even `[42 [0 1]]` is a cell. (It's very important to note that cells
can nest inside other cells.)

Collectively, both atoms and cells are called nouns. And Nock is just a list of
rules (or a set of patterns) for transforming nouns.  You put one noun in, you
get another noun out. It's like algebra: \( f(x) = y \), `nock(noun_a) = noun_b`

The atom `42` is a noun, the cell `[0 1]` is a noun and in fact `.*(42 [0 1])`
is a noun too, except that it's wrapped in syntax that tells Arvo "run
this noun through Nock". (The notation `.*(42 [0 1])` is actually the way to tell Arvo
(Urbit's operating system) to evaluate `Nock([42 [0 1]])`.) The cellular noun
`[42 [0 1]]` goes into Nock, the atomic noun `42` comes out.

We could simply write our very first example again as

```text
nock([42 [0 1]]) = 42
```
except that we wouldn't be able to evaluate it in Arvo.

Actually, we almost never use the above mathematical notation. For the sake of
brevity we almost always write `nock(a)` using the notation `*a`. Thus, `nock([42 [0 1]])` and `*[42 [0 1]]` mean the same thing. If we want to tell Arvo to evaluate
the noun, we use `.*(42 [0 1])`. You will notice that the outermost brackets
somewhat confusingly disappear when we use the `.*()` function in Arvo; we will
explain why in Chapter~\ref{cha:dont_know}.


In documenting Nock, we most frequently use the `*[42 [0 1]]` style of
notation. Instead of, for example, writing

```text
~tomsyt-balsen/try=> .*(a [1 b])
b
```

the standard way of writing out the rules of Nock is

```text
*[a [1 b]]                  b
```
with the left hand side being the noun that matches our input and the right
side being the product of that input.

We've been using `a` and `b` as variables that represent numbers (i.e., atoms),
but we can and do use them more broadly to represent nouns in general.

Let's apply the rule

```text
*[a [1 b]]                  b
```

which, to reiterate, means that when we run any noun of the form `[a [1 b]]`
through Nock (using the expression `.*(a [1 b])` in Arvo) always produces `b`,
regardless of `a`.

Let's run a few examples:

```text
~tomsyt-balsen/try=> .*(301 [1 374])
374

~tomsyt-balsen/try=> .*([42 43] [1 312])
312

~tomsyt-balsen/try=> .*(374 [1 [44 48]])
[42 43]

~tomsyt-balsen/try=> .*([46 49] [1 [456 539]])
[456 539]

~tomsyt-balsen/try=> .*(374 [1 [[[32 34] 33]]])
[[[31 32] 33]
```

### Summary
\label{sec:getting_started_summary}

To conclude this section, let's review what we've learned:

**Notation:**

Arvo syntax:

```text
~tomsyt-balsen/try=> .*(42 [0 1])
42
```

Math notation:

```text
nock([42 [0 1]]) = 42
```

Nock notation:

```text
*[42 [0 1]]                  42
```

**Error Messages:**

A syntax error occurs when an expression in Arvo is not typed in correctly.

```text
~tomsyt-balsen/try=> .*( 42 [0 1])
~ <syntax error at [1 4]>
```

The cell in the syntax error gives the line and column number of the location of the error.

```text
~tomsyt-balsen/try=> .*(42 [5 0 2])
! exit
```

**Structures:**

A noun is an atom or a cell.  An atom is a natural number.  A cell is an
ordered pair of two nouns, i.e., two atoms, two cells, or a cell and an atom.

**Nock Rules**

Lower-case letters such as `a` or `b` are variables that represent nouns.

Nock rules are notated with two columns, where the left hand side indicates
what pattern the noun matches, and the right hand side indicates what the noun
produces.
```text
*[a [0 1]]                  a
*[a [1 b]]                  b
```

### Exercises

1. Using the rule:

	```text
	*[a [0 1]]                  a
	```

	replace `a` with an atom, a cell, and multiple cells, and return them all respectively in Arvo (using, of course, the proper Arvo syntax).  


2. Using the rule:

	```text
	*[a [1 b]]                  b
	```
	Write a cell, substituting a noun of more then 4 cells for `b`, and any atom for `a`, and then return it in Arvo (again, in the correct syntax, of course). Then play around with what you substitute for a: change it to a different atom, make it a cell, and then a set of nested cells. Does it make a difference? If yes, why? If not, why not?

3. Produce a syntax error that reads:
```text
~ <syntax error at [1 14]>
```

4. With pen and paper, write out all three possible forms of notation (Arvo syntax, Math notation, and Nock notation) for each
of the expressions you evaluated in exercises 1. and 2.


## Noun Structure
\label{sec:noun_structure}

In Section~\ref{sec:getting_started} we played around with this:

```text
~tomsyt-balsen/try=> .*(42 [0 1])
42
```
which corresponds to the pattern
```text
*[a [0 1]]                  a
```

Now we're going to experiment with what happens when we replace `1` with
different values, as seen in Listing~\ref{code:several_nock_formulas}. (Remember, as you follow along, type out the examples at your own Arvo prompt.)


\begin{codelisting}
\label{code:several_nock_formulas}
\codecaption{Applying different formulas to the same argument.}
```text
~tomsyt-balsen/try=> .*([42 43] [0 1])
[42 43]

~tomsyt-balsen/try=> .*([42 43] [0 2])
42

~tomsyt-balsen/try=> .*([42 43] [0 3])
43

~tomsyt-balsen/try=> .*([42 43] [0 4])
! exit
```
\end{codelisting}

We don't really have enough examples in Listing~\ref{code:several_nock_formulas} to figure out the pattern yet. Let's
change `[42 43]` to `[[44 45] 43]` and try some more.
```text
~tomsyt-balsen/try=> .*([[44 45] 43] [0 1])
[[44 45] 43]

~tomsyt-balsen/try=> .*([[44 45] 43] [0 2])
[44 45]

~tomsyt-balsen/try=> .*([[44 45] 43] [0 3])
43

~tomsyt-balsen/try=> .*([[44 45] 43] [0 4])
44

~tomsyt-balsen/try=> .*([[44 45] 43] [0 5])
45

~tomsyt-balsen/try=> .*([[44 45] 43] [0 6])
! exit
```

It looks like anything of the form `*[a [0 b]]` produces the sub-nouns that are inside of `a`. Remember that `*[a [0 b]]` is the same as `.*(a [0 b])`, e.g., `a` is `[[44 45] 43]` and `b` is one of the atoms `1` through `6`.

But how does  `*[a [0 b]]` know which pieces of `a` to choose?

Let's do one more example and then we'll explain it.
```text
~tomsyt-balsen/try=> .*([42 [46 47]] [0 1])
[42 [46 47]]

~tomsyt-balsen/try=> .*([42 [46 47]] [0 2])
42

~tomsyt-balsen/try=> .*([42 [46 47]] [0 3])
[46 47]

~tomsyt-balsen/try=> .*([42 [46 47]] [0 4])
! exit

~tomsyt-balsen/try=> .*([42 [46 47]] [0 5])
! exit

~tomsyt-balsen/try=> .*([42 [46 47]] [0 6])
46

~tomsyt-balsen/try=> .*([42 [46 47]] [0 7])
47
```
Try to solve this puzzle on your own by playing with the following:
```text
~tomsyt-balsen/try=> .*(a [0 b])
```
where `a` is a cell and `b` is an atom. Try to pick atoms for `b` that are
small and try to pick cells for `a` that have lots of nesting.

When you're ready to have things explained to you, read on.

Think of a noun as a tree structure:
```text
  [42 [46 47]]
  /       \
42      [46 47]]
          / \
        46   47
```

Every cell has two branches (the head of the cell and the tail) leading down
from it. Atoms have no branches, because they can't be broken down any further
(which is why they're called atoms---from the Greek *\textgreek{ἄτομος}*, "indivisible").

Let's look at the tree of the other noun we played with, `[[44 45] 43]`:
```text
 [[44 45] 43]
    /      \
[44 45]    43
 /   \
44   45
```
It should be pretty obvious that we could change the values of any of the atoms
in the tree without changing the structure of the tree. That is to say, `[[44
45] 43]` and `[[24 25] 23]` have the same tree structure:
```text
 [[22 25] 23]
    /      \
[22 25]    23
 /   \
22   25
```
And now, for a more complicated tree, here's the noun `[[[48 49] 45] [46 47]]`:
```text
  [[[48 49] 45] [46 47]]
       /            \
 [[48 49] 45]      [46 47]
   /       \        /   \
[48 49]    45      46   47
 /   \
48   49
```
So how do the above trees relate to running `.*(a [0 b])`? Every part
of the tree gets mapped to an atomic address, called an *axis*. The mapping
looks something like this:
```text
           1
       /       \
     2           3
   /   \       /   \
  4     5     6     7
 / \   / \   / \   / \
8   9 10 11 12 13 14 15

```
Or, because the lines are kind of ugly:
```text
         1
    2          3
 4    5     6     7
8 9 10 11 12 13 14 15
```
Of course, this only a very small part of the entire tree. We extend the tree
by applying the rule: Every axis `/n` has a head with the axis`/2n` and a tail
with the axis `/2n+1`. +++recast & expand [MDH]+++

+++Is `/n` read "slash n"?+++

We map from noun to axis by comparing the tree of the noun with the axis tree
and seeing what matches. Like so, marking axes with a `/` character:
```text
  /1 : [[44 45] 43]
         /        \
 /2 : [44 45]     43 : /3
      /     \
/4 : 44      45 : /5
```
Again, because it bears repeating: the head of axis `/n` is `/2n` and the tail
of axis `/n` is `/2n+1`. Remember that the head is the left-hand noun and the
tail the right-hand noun of a cell-pair.

Start with 1. This is your root axis. All nouns have a valid axis `/1`, even
atoms. and the axis `/1` just refers to the noun itself. In the above example,
axis /1 of `[[44 45] 43]` is just `[[44 45] 43]`. The head of `[[44 45] 43]` is
`[44 45]` and the tail is `43`. Thus, axis `/2` of `[[44 45] 43]` is `[44 45]`
and axis /3 is `43`.

Another way to think about it is that the tree map has layers:
```text
layer 0             1
layer 1        2          3
layer 2     4    5     6     7
layer 3    8 9 10 11 12 13 14 15
```
which correspond to the nesting depth of the noun. If a noun is inside two
cells, like `46` inside `[[[48 49] 45] [46 47]]` then its axis is at layer 2 of
the tree. If its inside three cells like `48`, then its axis is at layer
three.

Recall the pattern we learned in Section~\ref{sec:getting_started_summary}:
```text
*[a [0 1]]                  a
```
This, we now see, is a special case of
```text
*[a [0 b]]              axis /b of a
```
The 0 in `*[a [0 b]]` is just an operator that means "axis". Nock maps simple
operators and functions to atoms, instead of a character like `/` for example,
because atoms (and cells) are all Nock knows. Fortunately for us,
there are only eleven atoms that are operators, atoms `0` through `10.`


### Summary

**Noun structure:**

Nouns are trees that look like this:
```text
 [[44 45] 43]
    /      \
[44 45]    43
 /   \
44   45
```
The left-hand of a cell is called the head. The right hand is the tail.

**Axes:**

An axis is the address of a node of the noun tree.

The notation for axis \(n\) is `/n`.

The first part of the axis tree looks like this:
```text
         1
    2          3
 4    5     6     7
8 9 10 11 12 13 14 15
```
The head of axis `/n` is `/2n` and the tail of axis `/n` is `/(2n+1)`.

**Nock Operators:**

The operators in Nock are functions mapped onto the eleven atoms `0` through `10`.

**Nock 0:**

The Nock operator that produces a given axis of a noun:
```text
*[a [0 b]]               /b of a
```

### Exercises

1. Take pen and paper and map out the axes of
```text
   [[[48 49] 45] [46 47]]
       /             \
 [[48 49] 45]      [46 47]
   /       \        /   \
[48 49]    45      46   47
 /   \
48   49
```
then to test yourself, run:
```text
~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [0 b])
```
for each axis `/b` of `[[[48 49] 45] [46 47]]`

2. Write a noun `a` such that the following produces something:
```text
~tomsyt-balsen/try=> .*(a [0 100])
```
In other words, find a noun that has an axis `/100`.

3. Prune your noun from the last exercise so that it's as short as possible,
while still having an axis `/100`.

4. If you still feel confused, replicate the previous two exercises with the
axes `/7`, `/17`, `/27`, `/47` and `/87`.

5. Build a noun that has every atom set to its own axis. The bigger the noun
the better. I'll get you started:
```text
1
[2 3]
[[4 5] 3]
[[4 5] [6 7]
```
To test different atoms in your noun, run
```text
~tomsyt-balsen/try=> .*(a [0 b])
```

## Nock 3
\label{sec:nock_3}

In the last section we learned how to access data inside nouns. Now we're going
to learn what you can do with data. After all, Nock is a computer, so you
should be able to, you know, compute things.

We mentioned in the last section that Nock has eleven operators, `0` through
`10`. We saw `Nock 1` in Section~\ref{sec:getting_started}:
```text
*[a [1 b]]                  b
```
We could describe `Nock 1` the constant operator, since it always produces `b`
no matter what you put in as `a`.

We also have `Nock 0`, which lets you refer to the sub-noun at the `axis b` of
your subject `a`.
```text
*[a [0 b]]               /b of a
```
We're going to skip `Nock 2` and cover it in Section~\ref{sec:nock_5_nock_2} and for now just jump
straight to `Nock 3`.

Before we do, let's quickly go over three new terms that'll help us talk about
operators:

Let's say you have the Nock expression
```text
*[a [1 b]]
```
We know that `1` is the operator, but shouldn't we have names for what `a` and
`b` are?

Let's call `a` the *subject*, because it's being subjected to our computation.
(Poor `a`.)

We'll call `b` an *argument*, and we'll call the cell `[1 b]` a *formula*.
Diagramming it out:
```text
*[a [1 b]]
*[subject [operator argument]]
*[subject formula]
```
Good, now that we've got vocab out of the way, let's get cooking.

Let's use the big noun `[[[48 49] 45] [46 47]]` from Section~\ref{sec:noun_structure} as our subject.
```text
~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 0])
! exit
```
Uh-oh, Arvo is telling us we just tried to do something indecent and unnatural.
```text
~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 1])
! exit

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 2])
! exit

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 3])
! exit

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 4])
! exit
```
Looks like `Nock 3` doesn't like an atomic argument. Let's go cellular.
```text
~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 [0 1]])
0
```
Okay, so at least that does something.

Let's try some more:
```text
~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 [0 2]])
0

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 [0 3]])
0

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 [0 4]])
0

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [3 [0 5]])
1
```
Wait, what?

Hold on, aren't `[0 1]` or `[0 5]` valid Nock formulas on their own?  If `3`
only takes cells, and formulas are cells, maybe it matters what the formula
does on its own.

Let's try that sequence again without `Nock 3`:
```text
~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [0 1])
[[[48 49] 45] [46 47]]

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [0 2])
[[48 49] 45]

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [0 3])
[46 47]

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [0 4])
[48 49]

~tomsyt-balsen/try=> .*([[[48 49] 45] [46 47]] [0 5])
45
```
One of these things is not like the others. Looks like something changes
whether the formula `[0 n]` refers to an atom or a cell in our subject.

Let's see what happens if we run Nock 3 with `[0 1]` as its argument and try
some different subjects:
```text
~tomsyt-balsen/try=> .*([42 42] [3 [0 1]])
0

~tomsyt-balsen/try=> .*(42 [3 [0 1]])
1

~tomsyt-balsen/try=> .*([1 1] [3 [0 1]])
0

~tomsyt-balsen/try=> .*(1 [3 [0 1]])
1
```
Looks like that's `0` for cells and `1` for atoms. Which means that `Nock 3` is
cell/atom tester.

How's that work?

`Nock 3`'s argument is itself a formula. Let's call it `b`.
```text
*[subject-a [3 formula-b]]
```
`Nock 3` produces either a `0` or a `1`, depending on what formula-b does when
applied to subject-a:
```text
*[subject-a formula-b]
```
Or more simply:
```text
*[a b]
```
If `*[a b]` produces a cell, then `*[a [3 b]]` produces 0. If `*[a b]` produces
an atom, `*[a [3 b]]` produces 1. So our pattern for Nock 3 is:
```text
*[a [3 b]]         ?(*[a b])
?(cell)            0
?(atom)            1
```
`?(x)` is just a little functional notation so that we can write out both possible branches.

We should also note that Urbit uses the atom `0` to mean "yes" and the atom `1`
to mean "no." (This may seem different and annoying, but it's
the same thing that Unix does.)

In that context, `Nock 3` is just asking the question "When the argument is applied to the subject, is the result a cell?"

### Summary

**Vocabulary**

- The **subject** is the noun that gets computed.

- The **operator** is an atom that tells us how to Nock the subject and arguments.

- The **arguments** modify the behavior of the operator.

- A **formula** is a cell of `[operator arguments]`

These can be summarized as follows:

```text
*[a [1 b]]
*[subject [operator arguments]]
*[subject formula]
```

**Yes and No**

Atom `0` means "yes"
Atom `1` means "no."

**Nock 3**

`Nock 3` is a cell tester.
```text
*[a [3 b]]         ?(*[a b])
?(cell)            0
?(atom)            1
```

### Exercises

1. Run and compare
```text
~tomsyt-balsen/try=> .*(a b)
```
and +++add a listing and a listing reference [MDH]+++
```text
~tomsyt-balsen/try=> .*(a [3 b])
```
with different values for `a` and `b`. +++add a listing and a listing reference [MDH]+++

2. Try finding a value for `b` that will return `1` if `a` is the atom `42`:
```text
~tomsyt-balsen/try=> .*(42 [3 b])
```
Not only is this possible, but you already know the formula to do it.

## Nock 4
\label{sec:nock_4}

Last section we learned how to test whether a noun is an atom or a cell with
`Nock 3`. Now we're going to figure out what `Nock 4` does.

Let's start playing with `Nock 4`, using `[[44 45] 46]` as our subject:
```text
~tomsyt-balsen/try=> .*([[44 45] 46] [4 0])
! exit

~tomsyt-balsen/try=> .*([[44 45] 46] [4 1])
! exit

~tomsyt-balsen/try=> .*([[44 45] 46] [4 2])
! exit
```
Okay, this is starting to feel a lot like `Nock 3`. Looks like `Nock 4` doesn't
like atoms either. Remember that `Nock 3` took a cell that was a formula as its
argument:
```text
*[subject [3 formula]]
```
And then depending on what the expression
```text
*[subject formula]
```
produced, `Nock 3` would return a `0` or `1`, according to the function `?(x)`
that we defined:
```text
*[a [3 b]]         ?(*[a b])
?(cell)            0
?(atom)            1
```
Let's assume that Nock 4 operates in a similar way. Let's use the formula `[0
n]` to try to apply Nock 4 to different axes in our subject:

```text
~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 1]])
! exit

~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 2]])
! exit

~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 3]])
47
```
Well! That's blessedly simple then. Watch:
```text
~tomsyt-balsen/try=> .*([[44 45] 46] [0 3])
46

~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 3]])
47
```
Can you guess what
```text
~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 4]])
```
would produce?

If you said
```text
~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 4]])
45
```
then you're starting to get the hang of this. +++recall from section ... that ... represents `/4`, which in the case of `[[44 45] 46]` is just `44`. Applying `Nock 4` to this yields `45`.+++


Yes, ladies and gentlemen, `Nock
4` is increment. Nock together your subject and the formula in your argument,
and whatever that produces, add 1 to it.

But what if `*[subject formula]` produces a cell instead of an atom? How do we
add `1` to a cell? Simple---we don't. The Sun continues to rise in the east,
pigs remain regretfully earthbound, Hell is still rather toasty, and
incrementing a cell in Nock produces an !\ exit:
```text
~tomsyt-balsen/try=> .*([[44 45] 46] [4 [0 2]])
! exit
```
Assuming we understood how `Nock 3` worked, writing down our pattern for `Nock 4` is easy:

Nock 3:
```text
*[a [3 b]]         ?(*[a b])
?(cell)            0
?(atom)            1
```
Nock 4:
```text
*[a [4 b]]         +(*[a b])
+(cell)            ! exit
+(atom)            1 + atom
```
`+(x)` is, again, just some notation so we can write out both branches of Nock
4.

An interesting property of Nock 4 is that  we can chain it together to
increment successive times.
```text
~tomsyt-balsen/try=> .*(44 [4 [0 1]])
45

~tomsyt-balsen/try=> .*(44 [4 [4 [0 4]]])
46

~tomsyt-balsen/try=> .*(44 [4 [4 [4 [0 4]]]])
47

~tomsyt-balsen/try=> .*(44 [4 [4 [4 [4 [0 4]]]]])
48

~tomsyt-balsen/try=> .*(44 [4 [4 [4 [4 [4 [0 4]]]]]])
49
```
Those brackets are starting to really pile up. Which is making this whole
process a lot less legible than we would like.

Fortunatley Nock has a notational rule that'll let us not have to write so many
of those brackets.
```text
~tomsyt-balsen/try=> .*(44 [4 [4 [4 [4 [4 [0 4]]]]]])
49

~tomsyt-balsen/try=> .*(44 [4 4 4 4 4 0 4])
49
```
Woah. That's a lot cleaner.

Concisely, Nock considers brackets to group to the right. If you'll recall from back in Section~\ref{sec:getting_started_summary}:
```text
A noun is an atom or a cell.
An atom is a natural number.
A cell is an ordered pair of two nouns.
```
Which means that formally, all nouns in Nock are either singletons (atoms) or
cells (pairs). There are no triples, quadruples, or \(n\)-[tuples](https://en.wikipedia.org/wiki/Tuple):
```text
[a b c]
[a b c d]
[a b c d e]
```
etc.

Although a triple like `[a b c]` doesn't exist in Nock, it is easier on the
eyes, so we can map a nested pair onto it:
```text
[a b c]         [a [b c]]
```
Which means that the Nock interpreter, whenever it sees a triple (or any
\(n\)-tuple, for \(n>2\)), just inserts the needed brackets.

Let's do some more examples to help you get the hang of it:
```text
~tomsyt-balsen/try=> .*(44 [4 [4 [4 [4 [4 0 1]]]]])
49

~tomsyt-balsen/try=> .*(44 [4 [4 [4 [4 4 0 1]]]])
49

~tomsyt-balsen/try=> .*(44 [4 [4 [4 4 4 0 1]]])
49

~tomsyt-balsen/try=> .*(44 [4 [4 4 4 4 0 1]])
49

~tomsyt-balsen/try=> .*(44 [4 4 4 4 4 0 1])
49
```

We can't get rid of the last pair, though:

```text
~tomsyt-balsen/try=> .*(44 4 4 4 4 4 0 1)
~ <syntax error at [1 18]>
```

This is just an artifact of Arvo's Nock interpreter, which we directly access
with the `.*` function (pronounced `dottar`, a contraction of "dot-star"). `.*` takes two
arguments, a subject and a formula:
```text
.*(subject [formula])
```
And for inscrutable reasons  +++recast this; what are the actual reasons, if any?+++, the formula has to be in brackets. As does the
subject, if it's a cell:
```text
~tomsyt-balsen/try=> .*(44 45 [4 4 4 4 4 0 2])
~ <syntax error at [1 9]>

~tomsyt-balsen/try=> .*([44 45] [4 4 4 4 4 0 2])
49
```

### Summary

Let's review:

**Nock 4:**
```text
*[a [4 b]]         +(*[a b])
+(cell)            ! exit
+(atom)            1 + atom
```
**Brackets:**

Brackets group to the right.
```text
[a b c]         [a [b c]]
```

**Nock interpreter:**

`.*` is pronounced "dottar"

Takes two arguments, subject and formula:
```text
.*(subject [formula])
```
formula must be bracketed.

### Exercises

1. Chain together Nock 4 and Nock 3, so that cells produce 2 and atoms produce 3.
2. Write a formula that always returns the cell [4 0 1].


## Nock 5, Nock 2 and Formula Distribution
\label{sec:nock_5_nock_2}

So we've learned how to do some simple operations with Nock. Now we're going to get a little fancier.


To jog your memory, we've seen the following operators so far: +++start adding massive repetition of what each `Nock n` is.+++

\begin{codelisting}
\label{code:nock_review}
\codecaption{A review of Nock 0, 1, 3, and 4.}
```text, options: "hl_lines": [4, 5, 10, 11, 18, 19, 25, 26]
~tomsyt-balsen/try=> :: Nock 0: Tree address
~tomsyt-balsen/try=> :: *[a [0 b]]               /b of a
~tomsyt-balsen/try=>
~tomsyt-balsen/try=>   .*([42 43] [0 1])
[42 43]

~tomsyt-balsen/try=> :: Nock 1: The constant operator
~tomsyt-balsen/try=> :: *[a [1 b]]               b for every a
~tomsyt-balsen/try=>
~tomsyt-balsen/try=>   .*([42 43] [1 0])
[42 43]

~tomsyt-balsen/try=> :: Nock 3: The cell tester
~tomsyt-balsen/try=> :: ?(cell)            0 ("yes")
~tomsyt-balsen/try=> :: ?(atom)            1 ("no")
~tomsyt-balsen/try=> :: *[a [3 b]]         ?(*[a b])
~tomsyt-balsen/try=>
~tomsyt-balsen/try=>   .*([42 43] [3 0 1])
0

~tomsyt-balsen/try=> :: Nock 4: Increment
~tomsyt-balsen/try=> :: +(cell)            ! exit
~tomsyt-balsen/try=> :: +(atom)            1 + atom
~tomsyt-balsen/try=> :: *[a [4 b]]         +(*[a b])
~tomsyt-balsen/try=>   .*([42 43] [4 0 2])
43
```
\end{codelisting}

These all have the pattern shown in Listing~\ref{code:formula_pattern}, where the operator is one of four atoms (0, 1, 3 or 4).

\begin{codelisting}
\label{code:formula_pattern}
\codecaption{}
```text
*[subject [operator arguments]]
```
\end{codelisting}


What if we replaced that atom with a cell? For example, in

```text
.*([42 43] [3 0 1])
```

let's see what happens if we replace the atom `3` with the cell `[0 1]`:


```text
~tomsyt-balsen/try=> .*([42 43] [[0 1] 0 1])
[[42 43] 42 43]
```

If we recall the `Nock 0` example from Listing~\ref{code:nock_review},

```text
~tomsyt-balsen/try=> .*([42 43] [0 1])
[42 43]
```

we see that `.*([[42 43] 42 43])` is just a nested combination of the result of the formula `.*([42 43] [0 1])`. In other words, it appears that Arvo is applying each operator in `[[0 1] 0 1]` to the argument `[42 43]` and combining the results.

Let's try some other formulas to see if this pattern holds (Listing~\ref{code:other_nested_formulas}).

\begin{codelisting}
\label{code:other_nested_formulas}
\codecaption{Applying a series of nested formulas to the same argument.}
```text
~tomsyt-balsen/try=> .*([42 43] [[0 2] 0 1])
[42 42 43]

~tomsyt-balsen/try=> .*([42 43] [[0 2] 0 2])
[42 42]

~tomsyt-balsen/try=> .*([42 43] [[0 1] 0 2])
[[42 43] 42]
```
\end{codelisting}


Recalling from Listing~\ref{code:several_nock_formulas} that

```text
~tomsyt-balsen/try=> .*([42 43] [0 2])
42
```

we see that the pattern we guessed continues to hold: Arvo is running both formulas and then just combining the results in a cell.

+++For the example below, we should establish the values of `.*([42 43] [3 0 2])` and `.*([42 43] [4 0 3])`+++

```text
~tomsyt-balsen/try=> .*([42 43] [[3 0 1] 3 0 2])
[0 1]

~tomsyt-balsen/try=> .*([42 43] [[3 0 1] 4 0 3])
[0 44]

~tomsyt-balsen/try=> .*([42 43] [[1 [0 1]] 4 0 3])
[[0 1] 44]

~tomsyt-balsen/try=> .*([42 43] [[4 0 3] 1 [0 1]])
[44 [0 1]]
```

Yup, the subject is definitely running through both formulas in parallel. The
last example seems to do something like this +++it doesn't match the example+++:

```text
*[[42 43] [4 0 3] 1 [55 73]]         [*[[42 43] [4 0 3]] *[[42 43] 1 [55 73]]]
```

which we can evaluate using Arvo:

```text
~tomsyt-balsen/try=> .*([42 43] [4 0 3])
44

~tomsyt-balsen/try=> .*([42 43] [1 [0 1]])
[0 1]
```

Or by hand, which is good practice. Open up a blank text file or grab a pen and
copy along:

```text
*[[42 43] [4 0 3] 1 [0 1]]  [*[[42 43] [4 0 3]] *[[42 43] 1 [0 1]]]

<<  Nock 4:       *[a [4 b]]          +(*[a b]) >>

[+(*[[42 43] 0 3]]) *[[42 43] 1 [0 1]]]

<<  Nock 0:       *[a [0 b]]          /b of a  >>

[+(43) *[[42 43] 1 [0 1]]]

<<     +():       +(atom)             1 + atom  >>

[44 *[[42 43] 1 [0 1]]]

<<  Nock 1:       *[a [1 b]]          b  >>

[44 [0 1]]
```
Thus,

```text
~tomsyt-balsen/try=> .*([42 43] [[4 0 3] 1 [0 1]])
[44 [0 1]]
```
We could write the first line of the reduction more generally as

```text
*[subject [formula1] formula2]     [*[subject formula1] *[subject formula2]]
```
This is a little long for my taste, but since a formula is a cell we can
rewrite it as in Listing~\ref{code:further_reduction}, where `a` is the subject, `[b c]` is `formula1` and `[d e]` is `formula2`.


\begin{codelisting}
\label{code:further_reduction}
\codecaption{}
```text
*[a [b c] d e]     [*[a b c] *[a d e]]
```
\end{codelisting}



Recalling the common pattern

```text
*[subject [operator arguments]]
```

from Listing~\ref{code:formula_pattern}, we see that in `*[a d]` the value of `d` must be a cell of the form `[operator arguments]`. This means that in Listing~\ref{code:further_reduction} we can relabel the cell `d e` (where `d` might be an atom) to just `d` (where now `d` has to be a cell), which yields Listing~\ref{code:final_reduction}.

\begin{codelisting}
\label{code:final_reduction}
\codecaption{}
```text
*[a [b c] d]     [*[a b c] *[a d]]
```
\end{codelisting}

What's really cool about the rule in Listing~\ref{code:final_reduction} that, like the operators, it also chains +++This does not appear to be of the form `*[a [b c] d]`+++:

```text
~tomsyt-balsen/try=> .*([42 [46 47]] [[0 1] [3 0 1] [0 2]])
[[42 46 47] 0 42]
```
So if we wanted to produce our subject with all the atoms incremented, we could
do that:

```text
~tomsyt-balsen/try=> .*([42 [46 47]] [[4 0 1] [4 0 2] [4 0 3]])
[43 47 48]
```
We can make our chains as long as we like

```text
~tomsyt-balsen/try=> .*([42 [46 47]] [[0 1] [3 0 1] [0 2] [3 0 2] [0 3]])
[[42 46 47] 0 42 1 [46 47] 0]
```

That is, we can evaluate arbitrary numbers of formulas on the same subject in
parallel.

But what if we want to run them in series?

The expression `*[[42 43] [[4 0 3] 1 [0 1]]]` is a good example of how this might work +++Make explicit in what sense this is evaluation in "series"+++:

```text
~tomsyt-balsen/try=> .*([42 43] [[4 0 3] 1 [3 0 1]])
[44 [3 0 1]]
```

Wouldn't it be interesting if we could run `[44 [3 0 1]]` through Nock again and end up with `*[44 [3 0 1]]` or just `1`? +++Why would this be interesting?+++

We'd need a recursive operator to do that +++why?+++. Fortunately, we've got one, Nock 2 +++this is the first explicit mention of Nock 2,and the example is rather complicated+++:

```text
~tomsyt-balsen/try=> .*([42 43] [2 [4 0 3] 1 [3 0 1]])
1
```
Obviously, this is a toy example because we could just do the same thing
functionally with +++this is not obvious—so far as I can tell, this is the first time that the second cell in a formula has been of length 4+++:

```text
~tomsyt-balsen/try=> .*([42 43] [3 4 0 3])
1
```
But Nock 2 also lets us call a formula inside our subject. +++I'm completely confused at this point+++

```text
~tomsyt-balsen/try=> .*([[40 43] [4 0 1]] [2 [0 4] [0 3]])
41

~tomsyt-balsen/try=> .*([[40 43] [4 0 1]] [2 [0 5] [0 3]])
44
```
Or we could completely separate the operator and arguments: +++Whaa?+++

```text
~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [2 [0 2] [0 31] [0 6] [0 30]])
44
```
We did a lot of slicing and dicing of nouns with the formula distribution rule.
Nock 2 lets us run those reassembled nouns as expressions. We could think of
Nock 2 as being exactly like the distribution rule:

```text
*[a [b c] d]          [*[a b c] *[a d]]
```
except that Nock 2 has an extra `*`, meaning we run everything through Nock a
second time. So if we have two formulas

```text
*[subject 2 formula1 formula2]      *[*[subject formula1] *[subject formula2]]
```
which we can rewrite as:

```text
*[a 2 b c]            *[*[a b] *[a c]]
```

Let's work through that last example again:

```text
~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [2 [0 2] [0 31] [0 6] [0 30]])
44
```

So we've got our subject `[[40 43] [0 1 3 4]]` and four different formulas:
`[0 2] [0 31] [0 6] [0 30]`

Let's apply each of these to our subject separately:

```text
~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [0 2])
[40 43]

~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [0 31])
4

~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [0 6])
0

~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [0 30])
3
```

If instead of Nock 2, we had just used the formula distribution rule:

```text
~tomsyt-balsen/try=> .*([[40 43] [0 1 3 4]] [[0 2] [0 31] [0 6] [0 30]])
[[40 43] 4 0 3]
```

But since Nock 2 is recursive:

```text
*[[[40 43] [0 1 3 4]] [2 [0 2] [0 31] [0 6] [0 30]]]
```
reduces to:

```text
*[[40 43] 4 0 3]
```
which is, of course, \(43 + 1\), or \(44\).

Now that we understand how to slice up nouns in our subject, let's introduce Nock 5.

Nock 5 is exactly like Nock 3 and Nock 4 in structure, but we've saved it for
last because it's easier to understand how to use it after you know how to
distribute formulas. See if you can figure it out from the following:

```text
~tomsyt-balsen/try=> .*([42 42] [5 [0 2] [0 3]])
0

~tomsyt-balsen/try=> .*([42 43] [5 [0 2] [0 3]])
1

~tomsyt-balsen/try=> .*([42 44] [5 [0 2] [0 3]])
1

~tomsyt-balsen/try=> .*([[42 42] [42 42]] [5 [0 2] [0 3]])
0

~tomsyt-balsen/try=> .*([[42 43] [42 43]] [5 [0 2] [0 3]])
0

~tomsyt-balsen/try=> .*([[42 43] [42 43]] [5 [0 4] [0 3]])
1

~tomsyt-balsen/try=> .*([[42 42] [42 42]] [5 [0 2] [0 3]])
0
```

Yes, Nock 5 is an equality test +++Instead of saying 'Yes', work through it in more detail+++:

```text
*[a 5 b]
```
If the head and the tail of the cell produced by `*[a b]` are the same, then
Nock 5 produces 0, if they are different, Nock 5 produces 1:

```text
*[a 5 b]              =(*[a b])
=([a a])              0
=([a !a])              1
```
Where `!a` just means "not a."

But what if `*[a b]` inside `=(*[a b])` produces an atom?

```text
~tomsyt-balsen/try=> .*([[42 42] [42 42]] [5 [1 1]])
! exit
```
So we need to add

```text
=(atom)              ! exit
```
to our rule.

Let's reduce the last example from above by hand:

```text
*[[[42 42] [42 42]] [5 [0 2] [0 3]]]

    *[a 5 b]                =(*[a b])

=(*[[[42 42] [42 42]] [0 2] [0 3]])

    *[a [b c] d]            [*[a b c] *[a d]]

=([*[[[42 42] [42 42]] [0 2]] *[[[42 42] [42 42]] [0 3]]])

    *[a [0 b]]               /b of a

=([[42 42] *[[[42 42] [42 42]] [0 3]]])

    *[a [0 b]]               /b of a

=([[42 42] [42 42]])

    =([a a])                 0

1
```

### Summary

**Formula Distribution:**

A formula with a second formula at its head instead of an operator distributes the subject over both formulas:
```text
*[a [b c] d]          [*[a b c] *[a d]]
```
`[b c]` is the first formula, d is the second formula.

**Nock 2**

```text
*[subject 2 formula-a formula]               *[*[subject formula1] *[subject formula2]]
```
which translates to

```text
*[a 2 b c]            *[*[a b] *[a c]]
```
**Nock 5**
```text
*[a 5 b]              =(*[a b])
=([a a])              0
=([a !a])              1
=(atom)               ! exit
```

### Exercises

1. Using the above rule, write a formula that reverses the order of the atoms in `[42 46 [68 69] 55]` (i.e., `[55 [68 69] 46 42]`).

2. Put the subject `[4 3 7 2 5 1 6]` in order from least to greatest.

3. Does `*[[42 42] 5 [0 1] [0 3]]` produce a `yes` or a `no`?

4. Write a noun that contains some data (nouns you find interesting) and some code (formulas you find interesting), write an expression with that noun as the subject that produces a single data-noun and a single code cell. Then use Nock 2 to apply the formula to the data.

5. Choose a subject such that the following expression evaluates

```text
~tomsyt-balsen/try=> .*(subject [2 [0 5] [0 4] [0 3]])
43
```

## Section VI: Conclusion

Let's list out all the rules we've learned so far, with the explanations collapsed:

```text
A noun is an atom or a cell.
An atom is a natural number.
A cell is an ordered pair of nouns.


[a b c]              [a [b c]]

?(cell)               0
?(atom)               1
+(cell)               ! exit
+(atom)               1 + atom
=([a a])              0
=([a !a])              1
=(atom)               ! exit


*[a 0 b]              /b of a
*[a 1 b]              b
*[a 2 b c]            *[*[a b] *[a c]]
*[a 3 b]              ?(*[a b])
*[a 4 b]              +(*[a b])
*[a 5 b]              =(*[a b])

*[a [b c] d]          [*[a b c] *[a d]]

```
That looks cleaner. Now let's look at an abridged version of the real Nock spec, which you should now be able to mostly read


```text
1  ::    A noun is an atom or a cell.
2  ::    An atom is a natural number.
3  ::    A cell is an ordered pair of nouns.
4  ::
5  ::    nock(a)          *a
6  ::    [a b c]          [a [b c]]
7  ::
8  ::    ?[a b]           0
9  ::    ?a               1
10 ::    +[a b]           +[a b]
11 ::    +a               1 + a
12 ::    =[a a]           0
13 ::    =[a b]           1
14 ::    =a               =a
15 ::
16 ::    /[1 a]           a
17 ::    /[2 a b]         a
18 ::    /[3 a b]         b
19 ::    /[(a + a) b]     /[2 /[a b]]
20 ::    /[(a + a + 1) b] /[3 /[a b]]
21 ::    /a               /a
22 ::
23 ::    *[a [b c] d]     [*[a b c] *[a d]]
24 ::
25 ::    *[a 0 b]         /[b a]
26 ::    *[a 1 b]         b
27 ::    *[a 2 b c]       *[*[a b] *[a c]]
28 ::    *[a 3 b]         ?*[a b]
29 ::    *[a 4 b]         +*[a b]
30 ::    *[a 5 b]         =*[a b]
31 ::
...
38 ::
39 ::    *a               *a
```

Since the Nock spec has been designed to be as concise as possible, a couple things are notationally different from the rules we've learned.

The largest difference is the block that defines what the `/` or axis operator does:

```text
16 ::    /[1 a]           a
17 ::    /[2 a b]         a
18 ::    /[3 a b]         b
19 ::    /[(a + a) b]     /[2 /[a b]]
20 ::    /[(a + a + 1) b] /[3 /[a b]]

```
This block of pseudocode is functionally equivalent to saying:

The head of axis /n is /(2n) and the tail of axis /n is /(2n+1).

If you read and understood Section~\ref{sec:noun_structure}, you understand what this is doing, even if you can't parse its recursive structure.

The next most obvious difference is that between

```text
8  ::    ?[a b]           0
9  ::    ?a               1
10 ::    +[a b]           +[a b]
11 ::    +a               1 + a
12 ::    =[a a]           0
13 ::    =[a b]           1
14 ::    =a               =a
```
and

```text
?(cell)               0
?(atom)               1
+(cell)               ! exit
+(atom)               1 + atom
=([a a])              0
=([a !a])             1
=(atom)               ! exit
```
We'll work through the evolution of this block:

First thing is that since we're trying to make the Nock specification small, we can get rid of the parentheses:

```text
?cell               0
?atom               1
+cell               ! exit
+atom               1 + atom
=[a a]              0
=[a !a]             1
=atom               ! exit

```
Then we can remove the words 'cell' and 'atom.' Since the rules in the nock spec match top to bottom, we can specify matching a rule to a cells or atom by putting [a b] above a. A cell will match [a b] and and atom will not, therefore because all cells will match [a b],
only atoms will match a below [a b].

```text
?[a b]             0
?a                 1
+[a b]             ! exit
+a                 1 + atom
=[a a]             0
=[a !a]            1
=a                 ! exit

```

Next little thing we can do, along the same principle, is change !a to b, because [a b] below [a a] will only match to a pair of unequal nouns.

```text
?[a b]             0
?a                 1
+[a b]             ! exit
+a                 1 + atom
=[a a]             0
=[a !a]            1
=a                 ! exit
```
Then there's these lines, which we call our crash defaults:

```text
10 ::    +[a b]           +[a b]
14 ::    =a               =a
21 ::    /a               /a
39 ::    *a               *a
```
Basically the crash defaults determine when Nock needs to return an ! exit, because something is nonsensical. In theory these lines imply that Nock spins forever in an infinite loop, in practice, Nock will just crash.

```text
10 ::    +[a b]           +[a b]

```
means that Nock crashes if you try to increment a cell

```text
14 ::    =a               =a
```
means that Nock crashes if you try run an equality test on an atom

```text
   21 ::    /a               /a
```
means that Nock crashes if you try to reference a noun axis that doesn't exist

```text
39 ::    *a               *a
```
means that if you try to run something that's not a valid formula (i.e. doesn't match any of the preceding lines 1 through 38) through Nock, you guessed it, Nock crashes.

Replacing ! exit in our rules with the appropriate crash default, we get the canoncial Nock specification:

```text
1  ::    A noun is an atom or a cell.
2  ::    An atom is a natural number.
3  ::    A cell is an ordered pair of nouns.
4  ::
5  ::    nock(a)          *a
6  ::    [a b c]          [a [b c]]
7  ::
8  ::    ?[a b]           0
9  ::    ?a               1
10 ::    +[a b]           +[a b]
11 ::    +a               1 + a
12 ::    =[a a]           0
13 ::    =[a b]           1
14 ::    =a               =a
15 ::
16 ::    /[1 a]           a
17 ::    /[2 a b]         a
18 ::    /[3 a b]         b
19 ::    /[(a + a) b]     /[2 /[a b]]
20 ::    /[(a + a + 1) b] /[3 /[a b]]
21 ::    /a               /a
22 ::
23 ::    *[a [b c] d]     [*[a b c] *[a d]]
24 ::
25 ::    *[a 0 b]         /[b a]
26 ::    *[a 1 b]         b
27 ::    *[a 2 b c]       *[*[a b] *[a c]]
28 ::    *[a 3 b]         ?*[a b]
29 ::    *[a 4 b]         +*[a b]
30 ::    *[a 5 b]         =*[a b]
31 ::
...
38 ::
39 ::    *a               *a

```

And that's it! That's really all there is to Nock. Everything else in Urbit, including the elided lines 32 through 38, is just a structure built on top of Nock. All your playing around in Urbit reduces to some combination of what you now already know.

