# Nock Is Easy

> \noindent *You get used to it. I don't even see the code.
> All I see is blonde, brunette, redhead.*
> \medskip \\
> ---**The Matrix**


##Fundamentals##

Now that we have all the tools, let's learn Nock from scratch.
Here are all the rules defining `*`:

```text
23 ::    *[a [b c] d]     [*[a b c] *[a d]]
24 ::
25 ::    *[a 0 b]         /[b a]
26 ::    *[a 1 b]         b
27 ::    *[a 2 b c]       *[*[a b] *[a c]]
28 ::    *[a 3 b]         ?*[a b]
29 ::    *[a 4 b]         +*[a b]
30 ::    *[a 5 b]         =*[a b]
31 ::
32 ::    *[a 6 b c d]     *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]
33 ::    *[a 7 b c]       *[a 2 b 1 c]
34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]
35 ::    *[a 9 b c]       *[a 7 c 2 [0 1] 0 b]
36 ::    *[a 10 [b c] d]  *[a 8 c 7 [0 3] d]
37 ::    *[a 10 b c]      *[a c]
38 ::
39 ::    *a               *a
```

As we saw in the last chapter, when `a` is an atom, `*a` is
always an error.  So Nock proper is a function of a cell.
Informally, that cell is always described as a pair

```text
[subject formula]
```

where `subject` is the data and `formula` is the program.  Notice
that `a` in the rules above, except 39, is always the subject.
So: let's learn how to write a Nock formula.

##Autocons##

We observe from the rules above that a formula, too, is always a
cell.  But when we look inside that cell, we see two basic kinds
of formulas:

```text
[operator operands]
[formula formula]
```

An operator is always an atom (`0` through `10`).  A formula is
always a cell.  Line 23 distinguishes these forms:

```text
23 ::    *[a [b c] d]     [*[a b c] *[a d]]
```

Suppose you have two formulas `f` and `g`, each of which computes
some function of the subject `s`.  You can then construct the
formula `h` as `[f g]`; and `h(s)` equals `[f(s) g(s)]`.

For instance, recall our computation from the last chapter:

```text
*[[19 42] [0 3] 0 2]
```

`s` is `[19 42]`, `f` is `[0 3]`, `g` is `[0 2]`, `h` is `[[0 3] 0
2]`.  `f(s)` is `42`; `g(s)` is `19`; so `h(s)` is `[42 19]`.

Why?  We could have an operator `11`, `cons` to a Lisp veteran,
with the operands `f` and `g`---so instead of writing

```text
[[0 3] 0 2]
```

we'd say

```text
[11 [0 3] 0 2]
```

But not only is this less elegant, it's less convenient.  Of
course, convenience at the Nock level matters little, but we
repeat this pattern at the Hoon level---where it's often more
pleasant to say `[a b]` than `(cons a b)`.

##Basic operators##

Nock is small, but it could be smaller.  If we didn't care at all
about the efficiency of the interpreter---in other words, if Nock
was a theoretical exercise rather than a practical tool---we
could make do with just the first six operators:

```text
25 ::    *[a 0 b]         /[b a]
26 ::    *[a 1 b]         b
27 ::    *[a 2 b c]       *[*[a b] *[a c]]
28 ::    *[a 3 b]         ?*[a b]
29 ::    *[a 4 b]         +*[a b]
30 ::    *[a 5 b]         =*[a b]
```

Let's run through them one by one.

###`0`###

`0` just applies the `/` function:

```text
25 ::    *[a 0 b]         /[b a]
```

For any subject `a`, the formula `[0 b]` produces `/[b a]`, which
is why

```text
*[[19 42] 0 3]
```

is `/[3 19 42]`, which is `42`.

###`1`###

`1` just ignores its subject and produces its operand:

```text
26 ::    *[a 1 b]         b
```

###`2`###

`2` is the only interesting basic operator:

```text
27 ::    *[a 2 b c]       *[*[a b] *[a c]]
```

Here we generate a calculation to perform.  Given the formula `[2
b c]`, `b` is a formula for generating the new subject; `c` is a
formula for generating the new formula.  To compute `*[a 2 b c]`,
we evaluate both `b` and `c` against the current subject `a`.

###`3`, `4`, `5`###

`3`, `4`, and `5` just apply `?`, `+` and `=` respectively---
that is, cell/atom, increment, and equals.

```text
28 ::    *[a 3 b]         ?*[a b]
29 ::    *[a 4 b]         +*[a b]
30 ::    *[a 5 b]         =*[a b]
```

##Macros##

Operators `6` through `10` are all essentially macros:

```text
32 ::    *[a 6 b c d]     *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]
33 ::    *[a 7 b c]       *[a 2 b 1 c]
34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]
35 ::    *[a 9 b c]       *[a 7 c 2 [0 1] 0 b]
36 ::    *[a 10 [b c] d]  *[a 8 c 7 [0 3] d]
37 ::    *[a 10 b c]      *[a c]
```

Each of these cases just resolves to another Nock computation, in
which each pattern matched on the left appears no more than once
on the right.  I.e., it's a macro.  But what do the macros do?
Let's work through them, from easiest to hardest.

###`10` (37)###

The second case of 10 is so easy it's puzzling:

```text
37 ::    *[a 10 b c]      *[a c]
```

For any `b`, the formula `[10 b c]` seems to be perfectly
equivalent to the formula... `c`.  But why?  Why would we say
`[10 b c]` when we could just say `c`?

The answer is that `10` is a hint to the interpreter.  It's true
that `[10 b c]` has to be *semantically* equivalent to `c`, but
it doesn't have to be *practically* equivalent.  Since whatever
information is in `b` is discarded, a practical interpreter is
free to ignore it, or to use it in any way that does not affect
the results of the computation.

###`7`###

`7` is our next easiest macro:

```text
33 ::    *[a 7 b c]       *[a 2 b 1 c]
```

Informally, the formula `[7 b c]` composes the formulas `b` and
`c`.  To use a bit of math notation, if `d` is `[7 b c]`,

```text
d(a) == c(b(a))
```

Let's see how this works by applying some reductions to the
definition of `7`, and producing a simpler definition that
doesn't look like a macro:

```text
*[a 2 b 1 c]

    <<27 ::    *[a 2 b c]       *[*[a b] *[a c]]>>

*[*[a b] *[a 1 c]]

    <<26 ::    *[a 1 b]         b>>

*[*[a b] c]
```

So we can write a revised line 33, perhaps slightly clearer:

```text
33r::    *[a 7 b c]       *[*[a b] c]
```

###`8`###

`8` looks slightly horrible but you shouldn't fear it at all:

```text
34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]
```

What does this even mean?  Let's go through the same process
of reducing it.

```text
*[a 7 [[7 [0 1] b] 0 1] c]

  <<33r::    *[a 7 b c]       *[*[a b] c]>>

*[*[a [7 [0 1] b] 0 1] c]

  <<23 ::    *[a [b c] d]     [*[a b c] *[a d]]>>

*[[*[a 7 [0 1] b] *[a 0 1]] c]

  <<33r::    *[a 7 b c]       *[*[a b] c]>>

*[[*[*[a 0 1] b] *[a 0 1]] c]

  <<25 ::    *[a 0 b]         /[b a]>>

*[[*[a b] a] c]
```

So our revised rule 34:

```text
34r::    *[a 8 b c]       *[[*[a b] a] c]
```

What does this actually do?  Well, look at it.  It evaluates the
formula `c` with the cell of `*[a b]` and the original subject
`a`.  In other words, in math notation, if `d` is `[8 b c]`,

```text
d(a) == c([b(a) a])
```

But why?  Suppose, for the purposes of `c`, we need not just `a`,
but some intermediate noun computed from `a` that will be useful
in `c`'s calculation.  We apply `c` with a new subject that's a
cell of the intermediate value and the old subject---not at all
unlike pushing a new variable on the stack.

For extra credit, a good question to ask yourself: why do we
need to write `[7 [0 1] b]` and not just `b`?

###`10` (36)###

We now understand all the moving parts we need to figure out the
other reduction of `10`:

```text
36 ::    *[a 10 [b c] d]  *[a 8 c 7 [0 3] d]
```

Reducing:

```text
*[a 8 c 7 [0 3] d]

  <<34r::    *[a 8 b c]       *[[*[a b] a] c]>>

*[[*[a c] a] [7 [0 3] d]]

  <<33r::    *[a 7 b c]       *[*[a b] c]>>

*[*[[*[a c] a] 0 3] d]
```

If you've assimilated a bit of Nock already, you may feel the
temptation to reduce this to

```text
*[a d]
```

since it would be very reasonable to think that

```text
*[[*[a c] a] 0 3]
```

is just `a`.  And it seems to be---given the semantics of 8 as
we've explained them.

But there's a problem, which is that `c` might not terminate.
If `c` terminates, this reduction is correct.  Otherwise...
it's not.  So the best we can do is:

```text
36r::    *[a 10 [b c] d]  *[*[[*[a c] a] 0 3] d]
```

Why?  `10` in either case is a hint.  If `x` in `[10 x y]` is an
atom, we reduce line 37 and `x` is simply discarded.  Otherwise,
`x` is a cell `[b c]`; `b` is discarded, but `c` is computed as a
formula and its result is discarded.

Effectively, this mechanism lets us feed both static and dynamic
information into the interpreter's hint mechanism.

###`6`###

`6` certainly looks intimidating:

```text
32 ::    *[a 6 b c d]     *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]
```

We could explain `6` as a reduction sequence.  But it's a long
one.  Instead, let's invent another operator which makes `6` easy:

       ::    $[0 b c]         b
       ::    $[1 b c]         c

Then we can restate `6` quite compactly:

```text
32r::    *[a 6 b c d]     *[a $[*[a b] c d]]
```

`6` stands revealed as the humble if-then-else.  Nock *is* easy.

This excuse for an explanation may not satisfy everyone.  A good
exercise is to check that `6` as defined *actually* has these
properties---and can't be simplified.

###`9`###

`9` is an audacious mystery:

```text
35 ::    *[a 9 b c]       *[a 7 c 2 [0 1] 0 b]
```

We'll reduce `9` but not explain it.  When we use it in an
example, it'll be obvious what it is.

```text
*[a 7 c 2 [0 1] 0 b]]

  <<33r::    *[a 7 b c]       *[*[a b] c]>>

*[*[a c] 2 [0 1] 0 b]]

  <<27 ::    *[a 2 b c]       *[*[a b] *[a c]]>>

*[*[*[a c] [0 1]] *[*[a c] 0 b]]

  <<25 ::    *[a 0 b]         /[b a]>>

*[*[a c] *[*[a c] 0 b]]
```

So we have:

```text
35r::    *[a 9 b c]       *[*[a c] *[*[a c] 0 b]]
```

If you have a really fine instinctive sense of Nock, you might
understand what `9` is for.  Otherwise, don't worry for now.