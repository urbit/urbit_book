# Nock Macros
\label{cha:nock_macros}

> \noindent *You get used to it. I don't even see the code.
> All I see is blonde, brunette, redhead.*
> \medskip \\
> ---**The Matrix**

In the last chapter we showed you the Nock specification with lines 32 through 37 removed.
This chapter will explain what these lines are and what they do:

```text
32 ::    *[a 6 b c d]     *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]
33 ::    *[a 7 b c]       *[a 2 b 1 c]
34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]
35 ::    *[a 9 b c]       *[a 7 c 2 [0 1] 0 b]
36 ::    *[a 10 [b c] d]  *[a 8 c 7 [0 3] d]
37 ::    *[a 10 b c]      *[a c]
```

These lines describe the Nock operators 6 through 10, which are just macros that reduce to operators 0 through 5. As you can see, these operators are defined in terms of Nock operators covered in the last chapter.

They add no new functionality to Nock, but we include them in the specification because they are essential to making Nock a practical basis for computation.

##Nock 7

We'll start with the easiest macro: Nock 7

```text
33 ::    *[a 7 b c]       *[a 2 b 1 c]
```
We actually already used Nock 7 in the last chapter while we were explaining Nock 2.

```text
~tomsyt-balsen/try=> .*([42 43] [2 [4 0 3] 1 [3 0 1]])
1
```
Which sequentially applies the formulas `[4 0 3]` and `[3 0 1]` to our subject `[42 43]`. We use Nock 7 whenever we evaluate two formulas on our subject in series. The Nock 1 operator in `*[a 2 b 1 c]` is, as we know, the constant operator, which means that when Nock 2 applies, as in the above example, '[1 [3 0 1]]` to the subject, [3 0 1] is produced.

Let's work through a full reduction of the above example, refer to the specification at the end of the last chapter to get a better sense of where these rules are coming from. And as always, please copy along on pen and paper or in a text file.

```text
*[[42 43] [2 [4 0 3] 1 [3 0 1]]]

      27 ::    *[a 2 b c]       *[*[a b] *[a c]]


*[*[[42 43] [4 0 3]] *[[42 43] 1 [3 0 1]]]

      26 ::    *[a 1 b]         b


*[*[[42 43] [4 0 3]] [3 0 1]]

      29 ::    *[a 4 b]         +*[a b]

*[+*[[42 43] 0 3] [3 0 1]]

      25 ::    *[a 0 b]         /[b a]

*[+/[3 [42 43]] [3 0 1]]

      18 ::    /[3 a b]         b

*[+43 [3 0 1]]

      11 ::    +a               1 + a

*[44 [3 0 1]]

      28 ::    *[a 3 b]         ?*[a b]

?*[44 0 1]

      25 ::    *[a 0 b]         /[b a]

?/[1 44]

      16 ::    /[1 a]           a

?44

      9  ::    ?a               1
1
```

This reduction is overly long because we've included the full reduction sequences for the Nock 0, 3 and 4 operators, which can usually be omitted without loss of clarity. The notation is often helpful, but must be balanced against the needs of conciseness.

Like so:

```text
*[[42 43] [2 [4 0 3] 1 [3 0 1]]]

    27 ::    *[a 2 b c]       *[*[a b] *[a c]]


*[*[[42 43] [4 0 3]] *[[42 43] 1 [3 0 1]]]

    26 ::    *[a 1 b]         b

*[*[[42 43] [4 0 3]] [3 0 1]]

    29 ::    *[a 4 b]         +*[a b]

*[44 [3 0 1]]

    28 ::    *[a 3 b]         ?*[a b]
1
```
The student of Nock should be able to fill in the missing steps in their minds.

Informally, the formula `[7 b c]` composes the formulas `b` and
`c`.  To use a bit of math notation, if `d` is `[7 b c]`,

```text
d(a) == c(b(a))
```
Let's see how this works by applying some reductions to the
definition of `7`, and producing a simpler definition that
doesn't look like a macro:


**`7` Reduction:**

```text
29 ::    *[a 7 b c]        *[a 2 b 1 c]

*[a 2 b 1 c]

23 ::    *[a 2 b c]        *[*[a b] *[a c]]

*[*[a b] *[a 1 c]]

22:    *[a 1 b]          b

*[*[a b] c]
```
**`7` Reduced:**

```text
7r ::     *[a 7 b c]         *[*[a b] c]
```
We'll skip the full reduction process for the rest of these macros and simply use the reduced forms to explain their function. Skip to the Nock reference guidechapter 4 if you want to see the full reduction sequence.

##Nock 8

```text
34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]

34r ::    *[a 8 b c]       *[[*[a b] a] c]
```

Nock 8 evaluates the formula `c` with the cell of `*[a b]` and the original
subject `a`.  In other words, in math notation, if `d` is `[8 b c]`,

```text
d(a) == c([b(a) a])
```
But why?  Suppose, for the purposes of `c`, we need not just `a`,
but some intermediate noun computed from `a` that will be useful
in `c`'s calculation.  We apply `c` with a new subject that's a
cell of the intermediate value and the old subject - not at all
unlike pushing a new variable on the stack.

Let's work through some examples:

```text
~tomsyt-balsen/try=> .*(42 [8 [4 0 1] [0 1]])
[43 42]

~tomsyt-balsen/try=> .*([42 45] [8 [[4 0 2] [4 0 3]] [0 1]])
[[43 46] 42 45]
```
For extra credit, a good question to ask yourself: In line 34,

```text
    34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]
```
why do we need to write `[7 [0 1] b]` and not just `b`?



##Nock 6

```text
32 ::    *[a 6 b c d]     *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]

32r ::    *[a 6 b c d]     *[a *[[c d] [0 *[[2 3] [0 ++*[a b]]]]]]
```

Actually, `6` is a primitive known to every programmer - good old
"if."  If `b` evaluates to `0`, we produce `c`; if `b` evaluates
to `1`, we produce `d`; otherwise, we crash.

For instance:

```text
~tomsyt-balsen/try=> .*(42 [6 [1 0] [4 0 1] [1 233]])
43
```
and

```text
~tomsyt-balsen/try=> .*(42 [6 [1 1] [4 0 1] [1 233]])
233
```

We can actually simplify the semantics of `6`, at the expense of
breaking the system a little, by creating a macro that works as
"if" only if `b` is a proper boolean and produces `0` or `1`.

This simpler "if" would be:

```text
32s::    *[a 6 b c d]    *[a 2 [0 1] 2 [1 c d] [1 0] [4 4 b]]
```
This reduces to

```text
32sr::   *[a 6 b c d]     *[a *[[c d] [0 ++*[a b]]]]
```
Let's describe what each of `a`, `b`, `c` and `d` are.

`a` is our subject, some data that we want to run our `if` on

`b` is our test formula, which returns a `yes` or a `no` when we apply it to our subject.

`c` is our `then` formula, which we want to apply to our subject if our test formula produces a `yes`, i.e. a 0.

`d` is our `else` formula, which we want to apply to our subject if our test formula produces a `no`, i.e. a 1.

Let's say we have the following Nock expression:

```text
~tomsyt-balsen/try=> .*(42 [6 [3 0 1] [4 0 2] [4 0 1]])
43
```
Our `test` is the formula [3 0 1], which tests if the subject is a cell.

Our `then` is the formula [4 0 2], which increments the head of a cell.

Our `else` is the formula [4 0 1], which increments an atom.

If we changed our subject to a cell:

```text
~tomsyt-balsen/try=> .*([40 43] [6 [3 0 1] [4 0 2] [4 0 1]])
41
```
With our simpler reduced if rule:

```text
32sr::   *[a 6 b c d]     *[a *[[c d] [0 ++*[a b]]]]
```
we could rewrite

```text
*[42 [6 [3 0 1] [4 0 2] [4 0 1]]]
```
as

```text
*[42  *[[[4 0 2] [4 0 1]] [0 ++*[42 [3 0 1]]]]]
```
Since `*[42 [3 0 1]]` produces a `no`, i.e a `1`:

```text
*[42  *[[[4 0 2] [4 0 1]] [0 ++1]]]]
```
which gets incremented twice
```text
*[42  *[[[4 0 2] [4 0 1]] [0 3]]]
```
and goes into a Nock 0 to select the tail of `[[4 0 2] [4 0 1]]`

```text
*[42 [4 0 1]]
```
which increments 42 to produce 43. You should be able to see how changing the subject to the cell `[40 43]` would cause a `0` to be produced by the test, and how that would cascade into the `then` formula `[4 0 2]` being selected and applied instead.

Our real `if` is only slightly more complicated:

```text
32r ::    *[a 6 b c d]     *[a *[[c d] [0 *[[2 3] [0 ++*[a b]]]]]]
```
There appears to be an extra step here, using Nock 0 twice, first to select from the cell [2 3], and then to select from the cell [c d].

The reason is fairly simple, if we just used the simpler version of `if`, tests that returned values other than 0 or 1 would have unexpected behaviour. If our our test produced a `3` we would then try to reference the axis /5 in our cell [c d]. Since the tail of `d` could very well be a valid formula, strange things could result.

We add the step of selecting from [2 3] because trying to reference anything other than /2 or /3 within [2 3] will crash, which is exactly what we want. (/1 of [2 3] won't crash, but this is fine since our test can never produce a -1)

##Nock 9

```text
9r ::     *[a 9 b c]        *[*[a c] *[*[a c] 0 b]]
```
We'll discuss what Nock 9 does in the next chapter, when we introduce how to
use _cores_, which are subjects containing both code and data.  If you have a
really fine instinctive sense of Nock, you might understand what `9` is for.

Succinctly, we use Nock 9 to call a formulas held inside of the subject itself,
and apply them to the subject. If you've been wondering how one writes Nock
expressions that loop, this is how.

##Nock 10

```text
36 ::    *[a 10 [b c] d]  *[a 8 c 7 [0 3] d]
37 ::    *[a 10 b c]      *[a c]
```
The second case of 10 is so easy it's puzzling:

```text
37 ::    *[a 10 b c]      *[a c]
```
For any `b`, the formula `[10 b c]` seems to be perfectly
equivalent to the formula `c`.  But why?  Why would we say
`[10 b c]` when we could just say `c`?

The answer is that `10` is a hint to the interpreter.  It's true
that `[10 b c]` has to be *semantically* equivalent to `c`, but
it doesn't have to be *practically* equivalent.  Since whatever
information is in `b` is discarded, a practical interpreter is
free to ignore it, or to use it in any way that does not affect
the results of the computation.

And the other reduction of `10`:

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

is just `a`.  And it seems to be - given the semantics of 8 as
we've explained them.

But there's a problem, which is that `c` might not terminate.
If `c` terminates, this reduction is correct.  Otherwise it's not.
So the best we can do is:

```text
36r::    *[a 10 [b c] d]  *[*[[*[a c] a] 0 3] d]
```

And that's it! That's the entirety of the Nock specification!

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
32 ::    *[a 6 b c d]     *[a 2 [0 1] 2 [1 c d] [1 0] 2 [1 2 3] [1 0] 4 4 b]
33 ::    *[a 7 b c]       *[a 2 b 1 c]
34 ::    *[a 8 b c]       *[a 7 [[7 [0 1] b] 0 1] c]
35 ::    *[a 9 b c]       *[a 7 c 2 [0 1] 0 b]
36 ::    *[a 10 [b c] d]  *[a 8 c 7 [0 3] d]
37 ::    *[a 10 b c]      *[a c]
38 ::
39 ::    *a               *a
```
