# Hoon Goes Wild

> *What good is a phone call if you're unable to speak?*
> \medskip \\
> ---**The Matrix**

##More simple types##

Before we actually do some programming with Hoon, let's meet two
more kinds of type---`%face` and `%fork`:

```text
++  type  $|  ?(%noun %void)
          $%  [%atom p=term]
              [%cell p=type q=type]
              [%cube p=* q=type]
              [%face p=term q=type]
              [%fork p=type q=type]
          ==
```

###`%fork`###

`%fork` is simply a union type.  A type is a set of nouns---
`[%fork p q]` means "it could be a p, or maybe a q."

Any branching computation in which different branches produce
different types will generate a fork.  For example, without
worrying too much about the mysterious `?:`:

```text
~waclux-tomwyc/try=> :type; ?:(& %foo [13 10])
%foo
{ %foo [@ud @ud] }

~waclux-tomwyc/try=> -:!>(?:(& %foo [13 10]))
[ %fork
  p=[%cube p=7.303.014 q=[%atom p=%tas]]
  q=[%cell p=[%atom p=%ud] q=[%atom p=%ud]]
]
```

Here we start to understand why the type renderer is useful, as
`{ %foo [@ud @ud] }` (which is *not* in any way Hoon syntax) is a
little easier to read than the actual type noun.

(Readers of a mathematical bent may ask: since Hoon has a union
type, where is the intersection type?  There is none.  Hoon is
not one of these languages whose goal is to be as mathematically
powerful as possible.  Since a programming language is a UI for
programmers, and programmers are not mathematicians, Hoon is
designed to be as powerful as it has to be---and no more.)

###`%face`###

A type is not just a set of nouns---it's also a semantics for
that set.  In practice, this means a namespace.

Let's use this feature:

```text
~waclux-tomwyc/try=> foo=42
foo=42
~waclux-tomwyc/try=> :type; foo=42
foo=42
foo=@ud
~waclux-tomwyc/try=> -:!>(foo=42)
[%face p=%foo q=[%atom p=%ud]]
```

With `%face`, we've simply wrapped a label around another type.
Note that this doesn't impair our ability to compute with the
value, which is of course the same noun it always was:

```text
~waclux-tomwyc/try=> (add 17 foo=42)
59
```

But how do we use this namespace?

To play comfortably with names, it'll help if we introduce some
Arvo shell syntax.  As in Unix, you can bind variables in the
Arvo shell:

```text
~waclux-tomwyc/try=> =test 42
~waclux-tomwyc/try=> test
42
~waclux-tomwyc/try=> (add 17 test)
59
```

(`=variable expression` is *not* in any way Hoon syntax---any
Hoon expression is a valid Arvo command, but not every Arvo
command is a Hoon expression.)

Let's put a `%face` inside this shell variable and try to use it:

```text
~waclux-tomwyc/try=> =test foo=42
~waclux-tomwyc/try=> test
foo=42
~waclux-tomwyc/try=> foo.test
42
```

You probably expected it to be `test.foo`.  This disoriented
feeling should vanish in a few minutes.  Let's go further:

```text
~waclux-tomwyc/try=> =test foo=42
~waclux-tomwyc/try=> test
foo=42
~waclux-tomwyc/try=> foo.test
42

~waclux-tomwyc/try=> =test bar=foo=42
~waclux-tomwyc/try=> test
bar=foo=42
~waclux-tomwyc/try=> -:!>(test)
[%face p=%bar q=[%face p=%foo q=[%atom p=%ud]]]

~waclux-tomwyc/try=> bar.test
foo=42
~waclux-tomwyc/try=> -:!>(bar.test)
[%face p=%foo q=[%atom p=%ud]]

~waclux-tomwyc/try=> foo.bar.test
42
~waclux-tomwyc/try=> -:!>(foo.bar.test)
[%atom p=%ud]

~waclux-tomwyc/try=> foo.test
! -find-limb.foo
! find-none
! exit
```

##Name resolution##

We're starting to learn a little about name resolution in Hoon.
We've seen that `foo.bar.test` means "foo in bar in test."  We've
seen that faces have to be unwrapped a layer at a time, so "foo in
test" is an error.

Let's try some cells:

```text
~waclux-tomwyc/try=> =test [cat=3 dog=4]
~waclux-tomwyc/try=> cat.test
3
~waclux-tomwyc/try=> =test [cat=3 dog=[pig=9 rat=12]]
~waclux-tomwyc/try=> rat.dog.test
12
```

We see that name resolution seeks into cells.  This solves one of
the problems we had when programming in Nock.  For example:

```text
~waclux-tomwyc/try=> =test [cow=97 test]
~waclux-tomwyc/try=> cow.test
97
~waclux-tomwyc/try=> rat.dog.test
12
```

By replacing `test` with `[cow=97 test]`, we've done exactly the
same thing as nock `8`.  (And we'll do more of it.)  Note that
because we didn't wrap a face around `test`, we seek into it when
looking for `dog`, and `rat.dog.test` works just the same way.
Even though `dog` is now at a different axis within `test`.

For reasons we'll see soon, we often want empty names.  As we saw
before, the syntax for an empty name is `$`.

```text
~waclux-tomwyc/try=> =test $=42
~waclux-tomwyc/try=> $.test
42
```

And interesting cases tell us more about the search algorithm:

```text
~waclux-tomwyc/try=> =test [cat=3 cat=[pig=9 rat=12]]
~waclux-tomwyc/try=> cat.test
3
~waclux-tomwyc/try=> pig.cat.test
! -find-limb.pig
! find-none
! exit
```

We see that when we search a cell, we search the head first.  It
is not in any way an error to have two faces with the same name.
And in fact, we can even work with this constraint:

```text
~waclux-tomwyc/try=> ^cat.test
[pig=9 rat=12]
~waclux-tomwyc/try=> pig.^cat.test
9
```

A `limb` to resolve is not just a name---it takes a prefix which
is an arbitrary number of `^` characters.  This count is the
number of name instances to ignore before matching.  For
instance:

```text
~waclux-tomwyc/try=> =test [cat=3 cat=[pig=9 rat=12] cat=42]
~waclux-tomwyc/try=> ^^cat.test
42
```

We're actually ready to describe the full resolution model...

###Wing resolution###

A `wing` is a dot-separated list, reading outside to in from
right to left.  Each element is a `limb`.  We've seen one kind of
limb---the name, with `^` prefixes.

But we can also use axes directly from Hoon.  For instance:

```text
~waclux-tomwyc/try=> =test [cat=3 dog=[pig=9 rat=12]]
~waclux-tomwyc/try=> +3.test
dog=[pig=9 rat=12]
~waclux-tomwyc/try=> dog.test
[pig=9 rat=12]
```

Note the difference between these two.  The noun is the same---
they are both `[9 12]`.  But the type is different:

```text
~waclux-tomwyc/try=> -:!>(+3.test)
[ %face
  p=%dog
    q
  [ %cell
    p=[%face p=%pig q=[%atom p=%ud]]
    q=[%face p=%rat q=[%atom p=%ud]]
  ]
]


~waclux-tomwyc/try=> -:!>(dog.test)
[ %cell
  p=[%face p=%pig q=[%atom p=%ud]]
  q=[%face p=%rat q=[%atom p=%ud]]
]
```

The axis gets us to the `%dog` face; the name actually removes it.
So we can write

```text
~waclux-tomwyc/try=> pig.dog.+3.test
9
~waclux-tomwyc/try=> pig.dog.test
9
~waclux-tomwyc/try=> pig.+3.test
! -find-limb.pig
! find-none
! exit
```

Perhaps this is obvious.  Perhaps it's not.

###Axis syntax###

This may seem like overkill.  Perhaps it *is* overkill.  But Hoon
has five syntaxes for an axis limb.

The first we've seen already: the axis itself as a decimal, e.g.,
`+3`.  The second is a simple dot, meaning `+1`:

```text
~waclux-tomwyc/try=> =test 42
~waclux-tomwyc/try=> ..test
42
```

Yes, that's the limb `.`, as applied (with `.`), to `test`.  Have
we gone crazy?  Perhaps---but in fact, this one gets used a lot.

Then we have an list-indexing syntax for constant offsets in
lists that (as is the Hoon convention) flow to the right.
Indices start at 1.  `&` produces the list element, `|` produces
the suffix:

```text
~waclux-tomwyc/try=> =test [1 2 3 4 ~]
~waclux-tomwyc/try=> &2.test
2
~waclux-tomwyc/try=> |2.test
[3 4 ~]
~waclux-tomwyc/try=> &1.test
1
~waclux-tomwyc/try=> |1.test
[2 3 4 ~]
```

This mechanism---which essentially just converts the list index
into an axis for `+`---is not used much, but nice when needed.
It applies only to constant indices, though, which is odd.  (For
non-constant indices, use the Hoon function `snag`.)

Finally, we have a graphical binary syntax which reads from left
to right, alternating the pairs `-`/`+` and `<`/`>` to mean head
and tail respectively.  For example:

```text
~waclux-tomwyc/try=> =test [[[8 9] [10 11]] [12 13] 14 30 31]
~waclux-tomwyc/try=> -.test
[[8 9] 10 11]
~waclux-tomwyc/try=> +.test
[[12 13] 14 30 31]
~waclux-tomwyc/try=> -<.test
[8 9]
~waclux-tomwyc/try=> +>.test
[14 30 31]
~waclux-tomwyc/try=> +>-.test
14
~waclux-tomwyc/try=> ->-.test
10
~waclux-tomwyc/try=> +>+<.test
30
```

The alternating glyphs create pleasant graphical patterns which
are moderately memorable when used in moderation.  Of course, in
general, when we have names we should use them.

###Resolving forks###

What happens when we resolve a name in a fork?  Yikes.  The
general principle is that name resolution across a fork works if,
and only if, the names resolve to the same axis on both branches.

For instance:

```text
~waclux-tomwyc/try=> =test ?:(& [pig=3 dog=4] [pig=%pig dog=%dog cat=%cat])
~waclux-tomwyc/try=> -:!>(test)
[ %fork
    p
  [ %cell
    p=[%face p=%pig q=[%atom p=%ud]]
    q=[%face p=%dog q=[%atom p=%ud]]
  ]
    q
  [ %cell
    p=[%face p=%pig q=[%cube p=6.777.200 q=[%atom p=%tas]]]
      q
    [ %cell
      p=[%face p=%dog q=[%cube p=6.778.724 q=[%atom p=%tas]]]
      q=[%face p=%cat q=[%cube p=7.627.107 q=[%atom p=%tas]]]
    ]
  ]
]
~waclux-tomwyc/try=> pig.test
3
~waclux-tomwyc/try=> -:!>(pig.test)
[%fork p=[%atom p=%ud] q=[%cube p=6.777.200 q=[%atom p=%tas]]]
```

And yet:

```text
~waclux-tomwyc/try=> dog.test
! -find-limb.dog
! find-fork
! exit
```

Why?  Because `dog` is at `+3` on one side of the fork, `+6` on
the other.

Frighteningly enough, we now have all the tools we need to really
start programming in Hoon...