# Using Nock

> \noindent *But are you crazy enough?*
> \medskip \\
> ---**Point Break**

##Playing with Nock##

Now we're going to actually do some cool stuff with Nock.

Fortunately, we have an entire OS, Arvo, which is built on Nock.
Unfortunately, there's really no practical reason to work
directly in Nock when you're using Arvo---except for learning
Nock, which you do once and never again.  So the things we'll
have to do are a little bit cumbersome.

What Arvo is good at is evaluating Hoon.  And it's possible to
evaluate Nock from Hoon, much the way you can put inline assembly
in C.  Through this indirection, we have two ways to run Nock in
Hoon: on the command line and via an app file.

###Command line###

From the Arvo command line, you can run one-liners with the Hoon
rune `.*`:

```text
~waclux-tomwyc/try=> .*(42 [4 0 1])
43
```

###Application file###

Unfortunately, the Arvo command line doesn't do multiline input
well, and if there's any hope of writing complex Nock formulas
it's by using plenty of whitespace and linebreaks.

So we've constructed a template for writing Nock formulas as Arvo
applications.  Unfortunately Arvo is a young OS and has no way to
edit a file.  But Arvo runs on Unix and Unix is a very old OS.
Arvo syncs its filesystem with your `$URBIT_HOME` directory,
propagating changes on either side.

Let's assume your `$URBIT_HOME` is `urb/`, and your ship is
`\sig waclux-tomwyc`.  The Nock application template is in

```text
urb/waclux-tomwyc/try/bin/nock.hoon
```

Its text should be:

```text
!:             ::  To write Nock as an Arvo application in Hoon
|=  *          ::
|=  [a=* ~]    ::  For educational purposes only
:_  ~  :_  ~   ::
:-  %la        ::  Preserve this mysterious boilerplate square
%+  sell  %noun::
.*  a          ::  Replace the formula with your own
:::::::::::::::::
               ::  Formula: increment
[4 0 1]
```

For the rest of this document we'll simply assume you can copy
boilerplate, and write the rest of the file:

```text
[4 0 1]                           ::    bump /1
```

(The pseudocode in the comments is not in any way described.  If
you have trouble figuring it out, that's okay, but you may not be
tall enough for the ride.)

Test this by running:

```text
~waclux-tomwyc/try=> :nock 42
43
```

Our first complex example will be a decrement function.  With or
without `vere` running, copy the template from Unix:

```text
$ cp urb/waclux-tomwyc/try/bin/nock.hoon urb/waclux-tomwyc/try/bin/dec.hoon
```

Then, use a Unix editor to change "Formula: increment" to
"Formula: decrement" in `dec.hoon`.

Either next time you start `vere`, or on your next keyboard event
if you're already running it, you'll see something like

     + /~waclux-tomwyc/try/1/bin/dec/hoon

Arvo has slurped up dec.hoon from your filesystem.  To test it,

```text
~waclux-tomwyc/try=> :dec 42
43
```

Well, we didn't change the formula, so it still increments.  But
it's a start.

##Decrement##

The only arithmetic operation in Nock is increment.  So how do we
decrement?  The algorithm is obvious: to decrement `n`, start
from `0`, and count up to `n-1`.  Or rather, count up to a number
`m` such that `m+1` equals `n`.

(Is this going to be an `O(n)` algorithm?  It is.  How do we
compute effectively in a platform where decrement is `O(n)`?
Gosh, it seems difficult, doesn't it?  We'll get to that.)

The first thing we're going to need is a counter.  Right now
our subject is just the atom we're trying to decrement---`/1`,
referenced with the formula `[0 1]`.  Thus, to increment it,
the formula is `[4 0 1]`.

Let's try to put the counter into the subject with one of our
macros operators, `8`.  Recall our revised rule for `8`:

```text
34r::    *[a 8 b c]       *[[*[a b] a] c]
```

The formula `c` is applied to the subject `[*[a b] a]`.  What is
our `b`?  It should just produce our initial counter value---0.
So, use operator `1` to produce a constant---`[1 0]`.  Let's
put this counter in the subject, and then increment as usual.

Edit `dec.hoon` so that the formula reads

```text
[ 8                               ::  push
  [1 0]                           ::    just 0
  [4 0 1]                         ::    bump /1
]                                 ::
```

Note that for these tall bracket structures, the space after `[`
is essential.  Then, you'll see the file automatically update in
Arvo:

```text
: /~waclux-tomwyc/try/2/bin/dec/hoon
~waclux-tomwyc/try=> :dec 42
```

Whoops!  It crashed:

```text
! /~waclux-tomwyc/try/~2013.11.26..00.01.38..499b/bin/dec/:<[4 1].[13 2]>
! /~waclux-tomwyc/try/~2013.11.26..00.01.38..499b/bin/dec/:<[4 8].[13 2]>
! /~waclux-tomwyc/try/~2013.11.26..00.01.38..499b/bin/dec/:<[5 1].[13 2]>
! /~waclux-tomwyc/try/~2013.11.26..00.01.38..499b/bin/dec/:<[6 1].[13 2]>
! /~waclux-tomwyc/try/~2013.11.26..00.01.38..499b/bin/dec/:<[7 1].[13 2]>
! exit
```

What did we do wrong?  We forgot that the subject had changed.
When we get to `[4 0 1]`, the subject is not `42`, but `[0 42]`---
the counter is there.  So our original argument, `42`, is
actually at `/3`:

```text
[ 8                               ::  push
  [1 0]                           ::   just 0
  [4 0 3]                         ::   bump /3
]                                 ::

: /~waclux-tomwyc/try/3/bin/dec/hoon
~waclux-tomwyc/try=> :dec 42
43
```

Okay, at least it increments again.  (Constantly readjusting tree
addresses by hand is one good reason to use a higher-level
language, like Hoon.) But now, perhaps, we can build a decrement
that works for at least one input value---`1`.

Obviously at some point we'll have to build a loop.  But for now,
all we need is an if statement that compares the incremented
counter to the original argument.  We know the original argument
is at `/3`, and the counter is at `/2`; we use the if operator,
`6`, and the equality test operator `5`.  If the comparison
fails, we shrug our shoulders and keep incrementing the argument.

```text
[ 8                               ::  push
  [1 0]                           ::   just 0
  [ 6                             ::   pick
    [5 [4 0 2] [0 3]]             ::    same (bump /2) /3
    [0 2]                         ::    /2
    [4 0 3]                       ::    bump /3
  ]                               ::   |
]                                 ::  |

: /~waclux-tomwyc/try/4/bin/dec/hoon
~waclux-tomwyc/try=> :dec 42
43
~waclux-tomwyc/try=> :dec 1
0
```

We're getting closer.  But now, that loop...

Up till now, our subject has contained only data.  If we want to
loop, we're obviously going to have to bite the bullet and put
code in our subject---which will become a `[code data]` cell.
In Nock (and Hoon) this is called a `core`.

Suppose we take our `6` formula and put it in the subject.  Then,
with this core subject `[formula counter argument]`, we'll run
the formula itself.  With this subject, the formula is `/2`, and
of course the core itself is `/1`.  So we can activate the core
with `[2 [0 1] [0 2]]`.

Of course, since the subject has changed again, we need to change
the addresses again.  The counter is now `/6` and the argument
is now `/7`:

```text
[ 8                               ::  push
  [1 0]                           ::   just 0
  [ 8                             ::   push
    [ 1                           ::    quid
      [ 6                         ::     pick
        [5 [4 0 6] [0 7]]         ::      same (bump /6) /7
        [0 6]                     ::      /6
        [4 0 7]                   ::      bump /7
      ]                           ::     |
    ]                             ::    |
    [2 [0 1] [0 2]]               ::    nock /1 /2
  ]                               ::   |
]                                 ::  |
```

This does exactly the same thing as before:

```text
: /~waclux-tomwyc/try/5/bin/dec/hoon
~waclux-tomwyc/try=> :dec 42
43
~waclux-tomwyc/try=> :dec 1
0
```

But somehow, we feel it *could* do better.  Why?  Because where
we do the useless `[4 0 7]`, we have a subject containing the
code we want to invoke.  It's just that the counter is wrong.

We need to do the same thing as `[2 [0 1] [0 2]`, but the subject
is not `[0 1]`.  That would be `[formula counter argument]`.  We
need `[formula (counter + 1) argument]`.

So, `formula` is `[0 2]`, `counter` is `[0 6]`, and `argument` is
`[0 7]`.  With autocons, we can just put them together to make a
(superfluous) formula for `[formula counter argument]`---i.e.,

```text
[[0 2] [0 6] [0 7]]               ::  cons /2 /6 /7
```

But we actually want to increment the counter:

```text
[[0 2] [4 0 6] [0 7]]             ::  cons /2 (bump /6) /7
```

And to invoke our formula on this modified core:

```text
[2 [[0 2] [4 0 6] [0 7]] [0 2]]   ::  nock (cons /2 (bump /6) /7) /2
```

If we put this into the decrement, it should actually work:

```text
[ 8                               ::  push
  [1 0]                           ::   just 0
  [ 8                             ::   push
    [ 1                           ::    quid
      [ 6                         ::     pick
        [5 [4 0 6] [0 7]]         ::      same (bump /6) /7
        [0 6]                     ::      /6
        [ 2                       ::      nock
           [[0 2] [4 0 6] [0 7]]  ::       (cons /2 (bump /6) /7)
           [0 2]                  ::       /2
        ]                         ::      |
      ]                           ::     |
    ]                             ::    |
    [2 [0 1] [0 2]]               ::    nock /1 /2
  ]                               ::   |
]                                 ::  |
```

And it does:

```text
: /~waclux-tomwyc/try/6/bin/dec/hoon
~waclux-tomwyc/try=> :dec 42
41
```

But there's one more step.  Remember operator `9`?

```text
35 ::    *[a 9 b c]       *[a 7 c 2 [0 1] 0 b]
35r::    *[a 9 b c]       *[*[a c] *[*[a c] 0 b]]
```

Suppose `c` is a formula that produces a core.  Then we see
immediately what `9` does: it activates a core, using the formula
at `/b` within the core.

So we can rewrite our decrement to use `9`:

```text
[ 8                               ::  push
  [1 0]                           ::   just 0
  [ 8                             ::   push
    [ 1                           ::    quid
      [ 6                         ::     pick
        [5 [4 0 6] [0 7]]         ::      same (bump /6) /7
        [0 6]                     ::      /6
        [9 2 [0 2] [4 0 6] [0 7]] ::      call.2 (cons /2 (bump /6) /7)
      ]                           ::     |
    ]                             ::    |
    [9 2 0 1]                     ::    call.2 /1
  ]                               ::   |
]                                 ::  |
```

Seems to work nicely:

```text
: /~waclux-tomwyc/try/6/bin/dec/hoon
~waclux-tomwyc/try=> :dec 42
41
```

Of course, there are limits:

```text
~waclux-tomwyc/try=> :dec 0
```

You'll have to hit `^C`, and you'll see a big ugly error stack.
Nock can work wonders but it can't decrement 0.  (Yes, you can
build signed integers in Hoon---they are represented as atoms
with the sign bit low.)

###A function###

As we start to build up toward language-level primitives, it
behooves us to do things the way a higher-level language would do
them.  Well, more exactly, the way Hoon does things.

Surprisingly, although a formula defines a function of the
subject, a function---at the language level---is not the same
thing as a formula.  Or rather, the argument is not the same
thing as the subject.

For instance, as we saw in decrement, the subject for the loop
needs to contain the code itself.  If we apply a formula which
can't call back into itself, our ability to loop is sorely
diminished.  So at the very least, when we call a function,
the subject can't just be `argument`---it has to be the cell
`[formula argument]`, so that the function can recurse.

Actually, it's confusing to say `argument`, because this implies
a special status for single and multiple arguments.  In Nock and
Hoon, we say `sample`, which is always one thing, but can be a
cell for "functions of two arguments", a triple for three, etc.
Eg, the sample for a decrement function is an atom; the sample
for an add function is a cell of two atoms; etc.

Furthermore, a function needs more data than just the argument---
it might, for instance, want to call other functions.  Where's it
going to get them?  There is no external environment in Nock.

So the standard convention for a Nock function---or a Hoon
function---is

```text
[formula sample context]
```

Where `formula` is the code, `sample` is the argument(s), and
`context` is any other data and/or code that may be useful.

It's a bit irregular that we are taking the external subject
and using it directly from our formula.  Let's try to build a
function with this convention and call it directly.

First, we'll build an increment function to keep things simple.
We actually don't need anything in the context, so we'll put 0.

```text
[ 8                               ::  push
  [                               ::   cons
    [1 [4 0 6]]                   ::    quid bump /6  ::  formula
    [1 0]                         ::    just 0        ::  sample
    [1 0]                         ::    just 0        ::  context
  ]                               ::   |
  [ 9                             ::   call
    2                             ::    .2
    [0 4] [0 3] [0 11]            ::    cons /4 /3 /11
  ]                               ::   |
]                                 ::  |

```

Why `[[0 4] [0 3] [0 11]]`?  Our goal in calling the function is
to take the blank default core we've created at `/2`, and
substitute in the original subject of the outer formula, which
before the outer `8` was `/1` and is now `/3`.  Around this
we wrap the formula from the default core, at `/4`, and the
(dummy) context, at `/11`---that is, `/7` within `/2`.

Let's fit our decrement into this framework:

```text
[ 8                                     ::  push
  [                                     ::   cons
    [ 1                                 ::    quid    ::  formula
      [ 8                               ::     push
        [1 0]                           ::      just 0
        [ 8                             ::      push
          [ 1                           ::       quid
            [ 6                         ::        pick
              [5 [4 0 6] [0 30]]        ::         same /6 /30
              [0 6]                     ::         /6
                [9 2 [0 2] [4 0 6] [0 7]] ::         call.2 /2 (bump /6) /11
            ]                           ::        |
          ]                             ::       |
          [9 2 0 1]                     ::       call.2 /1
        ]                               ::      |
      ]                                 ::     |
    ]                                   ::    |
    [1 0]                               ::    just 0  ::  sample
    [1 0]                               ::    just 0  ::  context
  ]                                     ::   |
  [9 2 [0 4] [0 3] [0 11]]              ::   call.2 /4 /3 /11
]                                       ::  |
```

Observe that nothing has changed from the way we called our
increment function, and only one thing has changed within the
decrement formula---the axis of the argument.  Now at `/7` is not
the naked argument to decrement, but our outer core.  The sample
is at `/6` within this `/7`, i.e., at `/30`.

##A library##

Frankly, this is getting close to the limits of anything you'd
want to do in hand-generated Nock.  But why not press on?

What we'd really like to do is build a library of functions that
can call each other.  It's easy to guess that this library will
be... a core.  But what does this core look like?

A function core, `[formula sample context]`, is a very useful
kind of core, but it's not the only kind of core.  (Actually,
because the word "function" is too easy to throw around, we have
a special name for a function core: we call it a `gate`.  Compare
to "lambda" or "closure.")

But in general, a core is just `[code data]`---or, to use more
lingo, `[battery payload]`.  The payload can be anything---it's
just data.

The battery can be one *or more* formulas, each of which is
applied with the core as its subject.  This is why `9` takes the
axis operand `b`.  If the core is a gate, the battery is just one
formula; this is the head of the core, so `b` is 2.

But not every core is a gate.  Suppose we want to build a
library?  We could assemble a bundle of cores and put it in
the context.  So, let's say we need to write subtract, which
obviously is going to use decrement.  So, the context will be

  [subtract-gate decrement-gate]

But wait.  Each gate is [formula sample context].  So, because
Nock doesn't do cycles, there's no way the subtract gate and the
decrement gate can each reference each other through the context.
It happens to be the case here that subtract needs decrement, but
decrement doesn't need subtract.  But we're not looking for ugly
at this point---we know Nock is more than capable of that.

To support general mutual recursion, our library needs to be a
battery in which each formula produces a gate.  The context of
that gate is the library core.

Let's repeat this again because it's so important.  Our library
will be a battery in which each formula produces a gate.  The
context of that gate is the library core.

Let's build a trivial library core of this form, with one
function, good old increment.  Then, we'll call it.

```text
[ 8                               ::  push
  [                               ::   cons
    [ 1                           ::    quid          ::  battery
      [1 [4 0 6]]                 ::     quid bump /6
      [1 0]                       ::     just 0
      [0 1]                       ::     /1
    ]                             ::    |
    [1 0]                         ::    just 0        ::  payload
  ]                               ::   |
  [ 8                             ::   push
    [9 2 0 2]                     ::    call.2 /2
    [9 2 [0 4] [0 7] [0 11]]      ::    call.2 /4 /7 /11
  ]                               ::   |
]                                 ::  |
```

Compare this to the standalone increment above.  It's obviously
more complex and it should be.

First of all, what we put in the library core is not the function
gate directly, but a formula that generates the gate.  This way,
and only this way, we can put the library itself in the context.

Second, what's the payload of the library core?  It's `0`,
because the library doesn't depend on anything.  It certainly
doesn't depend on the argument to our application.

Third, now we can't just call the gate directly.  We have to
actually build it.  So we need another `8` to "push it on the
stack", and then we call it with the usual `9`.  Since the
subject at this point is `[gate library argument]`, the sample we
use is `[0 7]` rather than `[0 3]`---everything else is the same.

But does it work?  C'mon, you know it works:

```text
~waclux-tomwyc/try=> :dec 42
43
```

Okay, let's go ahead and put our actual decrement function in
the library.  We won't write the pseudocode here, because it's an
excellent exercise to add it---see below.

```text
[ 8
  [
    [
      [ 1
        [ 1
          [ 8
            [9 5 0 7]
            [ 6
              [5 [1 0] [0 29]]
              [0 28]
              [ 9
                2
                [0 6]
                [ [9 2 [0 4] [0 28] [0 15]]
                  [9 2 [0 4] [0 29] [0 15]]
                ]
                [0 15]
              ]
            ]
          ]
        ]
        [1 0]
        [0 1]
      ]
      [ 1
        [ 1
          [ 8
            [1 0]
            [ 8
              [ 1
                [ 6
                  [5 [4 0 6] [0 30]]
                  [0 6]
                  [9 2 [0 2] [4 0 6] [0 7]]
                ]
              ]
              [9 2 0 1]
            ]
          ]
        ]
        [1 0]
        [0 1]
      ]
    ]
    [1 0]
  ]
  [ 8
    [9 4 0 2]
    [ 9
      2
      [0 4] [0 7] [0 11]
    ]
  ]
]
```

Then, let's go crazy and add a subtract function, which calls
decrement.

```text
[ 8
  [
    [
      [ 1
        [ 1
          [ 8
            [9 5 0 7]
            [ 6
              [5 [1 0] [0 29]]
              [0 28]
              [ 9
                2
                [0 6]
                [ [9 2 [0 4] [0 28] [0 15]]
                  [9 2 [0 4] [0 29] [0 15]]
                ]
                [0 15]
              ]
            ]
          ]
        ]
        [1 0]
        [0 1]
      ]
      [ 1
        [ 1
          [ 8
            [1 0]
            [ 8
              [ 1
                [ 6
                  [5 [4 0 6] [0 30]]
                  [0 6]
                  [9 2 [0 2] [4 0 6] [0 7]]
                ]
              ]
              [9 2 0 1]
            ]
          ]
        ]
        [1 0]
        [0 1]
      ]
    ]
    [1 0]
  ]
  [ 8
    [9 4 0 2]
    [ 9
      2
      [0 4] [0 7] [0 11]
    ]
  ]
]
```

Note that the call to build the gate is `[9 4 0 2]`, because the
subtract arm is the head of the battery, which is the head of the
core---i.e., `/2` within `/2`---i.e., `/4`.

Does this work?  Really?

```text
~waclux-tomwyc/try=> :dec [42 12]
30
```

##Exercises##

Do you actually know Nock now?  Well, possibly.

A good exercise is to add more simple math functions to this
battery.  Try add, multiply, and divide.  One way to start is by
walking through the uncommented routines above, putting
pseudocode comments on them, and figuring out what they're doing.

Computing axes is slightly arduous (which is why we use Hoon,
generally).  We are torturing ourselves by using Nock, but we
might as well use Hoon to calculate axes:

```text
~zod/try=> (peg 3 3)
7
~zod/try=> (peg 3 5)
13
```

I.e., `(peg a b)` is `/b` within `/a`.  Writing Nock without this
would be pretty tough...