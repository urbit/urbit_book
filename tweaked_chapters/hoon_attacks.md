# Hoon Attacks

> \noindent *Como todos los hombres de Babilonia, he sido procónsul;
> como todos, esclavo\ldots*
> \bigskip \\
> \noindent *Like all men in Babylon, I have been proconsul;
> like all, a slave\ldots*
> \medskip \\
> ---[**_La lotería en Babilonia_**](https://en.wikipedia.org/wiki/The_Lottery_in_Babylon), Jorge Luis Borges

##Principles of type##

By working through the Nock tutorial, you've actually come closer
than you know to knowing Hoon.  Hoon is actually not much more
than a fancy wrapper around Nock. People who know C can think of
Hoon as the C to Urbit's Nock---just a sprinkling of syntax,
wrapped around machine code and memory.

For instance, it's easy to imagine how instead of calculating
tree axes by hand, we could actually assign *names* to different
parts of the tree---and those names would stay the same as we
pushed more data on the subject.  It can't hurt to dream, right?

The way we're going to do this is by associating something called
a `type` with the subject.  You may have heard of types before.
Technically, Hoon is a statically typed language, which just
means that the type isn't a part of your program: it's just a
piece of data the compiler keeps around as it turns your Hoon
into Nock.

In some languages, especially functional languages, types are
dangerous scary concepts that involve a lot of math.  For those
who like this sort of thing, that's the sort of thing they like.
For the rest of us, there's Hoon. It's a functional language, but
not one of those scary ones.

A lot of other languages use dynamic types, in which the type of
a value is carried along with the data as you use it.  Even
languages like Lisp, which are nominally typeless, look rather
typed from the Hoon perspective.  For example, a Lisp atom knows
dynamically whether it's a symbol or an integer.  A Hoon atom is
just a Nock atom, which is just a number.  So without a static
type, Hoon doesn't even know how to print an atom properly.

When we parse a Hoon expression, file, etc, we produce what we
call a `hoon`, which (if you know the CS jargon) is an AST.  A
hoon is a noun that's converted into a Nock formula, with
the assistance of a type which describes the subject of the
formula:

```text
[subject-type hoon] => formula
```

But actually this isn't quite right, because Hoon does something
called "type inference."  When we have a type that describes the
subject for the formula we're trying to generate, as we generate
that formula we want to also generate a type for the product of
that formula on that subject.  So our compiler computes:

```text
[subject-type hoon] => [product-type formula]
```

As long as `subject-type` is a correct description of some
subject, you can take any `hoon` and compile it against
`subject-type`, producing a `formula` such that `*(subject
formula)` is a product correctly described by `product-type`.

Actually, this works well enough that in Hoon there is no direct
syntax for defining or declaring a type.  There is only a syntax
for constructing hoons.  Types are always produced by inference.

##Printing types##

Let's start looking at types with the simplest possible kind of
hoon---an atomic constant, which ignores the subject and its
type, and just produces its own subject.  Here's everyone's
favorite atomic constant:

```text
~waclux-tomwyc/try=> 42
42
```

Let's also print its type---in two different ways:

```text
~waclux-tomwyc/try=> :type; 42
42
@ud

~waclux-tomwyc/try=> -:!>(42)
[%atom p='ud']
```

W-what?  Since types are of course static, printing them out
dynamically involves a little bit of black magic at the Hoon
and/or Arvo levels.  All will be revealed in due course.

But a type, like everything else in Hoon, is a noun.  Suffice it
to say that `-:!>(42)` is just printing this noun, whereas
`:type; 42` is rendering it intelligently as a string.  In this
case, the rendering is actually Hoon syntax, but in general it's
just a string.

In this case, looking directly at the type noun is preferable.
But for a core, the type actually contains the entire codebase.
It is no problem to compute with this, but we can't look at it
without scrolling more or less to infinity.

##Atom examples##

Let's look at some more of these atoms:

```text
~waclux-tomwyc/try=> :type; 0x42
0x42
@ux

~waclux-tomwyc/try=> :type; 'foo'
'foo'
@ta

~waclux-tomwyc/try=> :type; ~2013.12.6
~2013.12.6
@da

~waclux-tomwyc/try=> :type; .127.0.0.1
.127.0.0.1
@if

~waclux-tomwyc/try=> :type; ~m45
~m45
@dr

~waclux-tomwyc/try=> :type; `@da`(add ~2013.12.6 ~m45)
~2013.12.6..00.45.00
@da
```

Okay, we got a little out of control with that last one.  But the
point should be clear.

Note again that there is no dynamic type here.  All these values
are atoms.  Let's cast them all to decimals to see (don't worry
about the cast syntax---we'll cover that later):

```text
~waclux-tomwyc/try=> `@ud`0x42
66
~waclux-tomwyc/try=> `@ud`'foo'
7.303.014
~waclux-tomwyc/try=> `@ud`.127.0.0.1
2.130.706.433
~waclux-tomwyc/try=> `@ud`~2013.12.6
170.141.184.500.724.667.905.957.736.036.171.776.000
~waclux-tomwyc/try=> `@ud`~m45
49.806.208.999.015.789.363.200
```

(Yes, 45 minutes is actually quite a large number when your unit
of time is \( 2^{-64} \) seconds.)

What are these atoms, anyway?  Let's look at the actual type noun
(which, without magic, exists only at compile time):

```text
~waclux-tomwyc/try=> -:!>(42)
[%atom p='ud']
```

`p` is called the `odor` of the atom.  In this case, it's `'ud'`,
i.e., unsigned decimal:

```text
~waclux-tomwyc/try=> `@ud`'ud'
25.717
```

That's an unsigned-decimal interpretation of the atom 'ud', which
without a cast is an atom of odor `@ta`:

```text
~waclux-tomwyc/try=> :type; 'ud'
'ud'
@ta

~waclux-tomwyc/try=> -:!>('ud')
[%atom p='ta']
```

In case this is at all mysterious, we note:

```text
~waclux-tomwyc/try=> (add 'u' (mul 256 'd'))
25.717
```

As a general convention, when we represent UTF-8/ASCII text as an
atom, we store it LSB first.  A text atom is generally called a
`cord`; if it is ASCII only, a `span`; if it is ASCII restricted
to the Hoon symbol subset (lowercase with hyphens), a `term`.

##The logic of smell##

What is an odor, anyway?  We've seen that the type of an atom
constant gets an odor which is a function of its syntax.  We've
also seen that when we print atoms on the console, the printer is
(in some black-magic way) aware of the odor, and can print the
atom in roughly the same syntax we originally entered it.

Hoon's type system is generally strong, but odors are weak.  The
parser itself will only generate a `@ta` for an actual ASCII
span, but not every atom is a constant.  For instance, consider
our construction of `'ud'`.  Let's look at the type:

```text
~waclux-tomwyc/try=> :type; 'u'
'u'
@ta

~waclux-tomwyc/try=> :type; 256
256
@ud

~waclux-tomwyc/try=> :type; (add 'u' (mul 256 'd'))
25.717
@
```

Not only are we mixing atoms of different odors in our
arithmetic, but the result appears to be... odorless.  It is
odorless.  But we can cast it back:

```text
~waclux-tomwyc/try=> :type; `@ta`(add 'u' (mul 256 'd'))
'ud'
@ta
```

While Hoon's type system is quite intelligent in some ways, it is
by no means smart enough to evaluate your arithmetic and conclude
that it produces a valid ASCII span.  We can convert any atom to
any other odor of atom, without any sanity checks:

```text
~waclux-tomwyc/try=> `@dr`25.717
~.s0..0000.0000.0000.6475

~waclux-tomwyc/try=> `@if`25.717
.0.0.100.117
```

It so happens that `25.717` is a valid amount of time, and also a
valid IPv4 address.  But obviously not all atoms are meaningful
values of every odor.  We're not going to annoy you by stressing
out the console printer with bad ASCII, but we could.

(And why do we say `25.717` rather than `25,717`?  Are we...
Germans?  No, but we want all atom syntaxes to be URL-safe.
See below.)

Odors are a weak type system because the programmer often knows,
at a logical level not at all available to the type system, that
(for example) `(add 'u' (mul 256 'd'))` produces an ASCII span.
We want to keep the programmer from accidentally using a date as
if it were text, but we don't want to keep her from intentionally
converting between odors or ascribing odor to the result of an
arbitrary computation.

An odor is exactly what it looks like---an ASCII span.  This span
is a taxonomy which grows more specific to the right.  For
instance, `@t` for UTF-8 text, `@ta` for URL-safe ASCII text,
`@tas` for a Hoon symbol; or `@u` for an unsigned integer, `@ux`
for an unsigned integer formatted as hexadecimal.

The general principle of type enforcement is that atoms change
freely either up or down the taxonomy, but not across.  For
instance, you can treat a `@tas` as a `@t`, as in a strong type
system; but you can also treat a `@t` as a `@tas`, or an `@` as
anything.  However, passing a `@t` to a function that expects an
`@ux` is a type error.

Even the ability to cast a `@ud` to a `@dr` is a syntactic hack;
casts in Hoon do not evade type enforcement.  When you write

```text
`@dr`25.717
```

the parser actually produces the equivalent of

```text
`@dr``@`25.717
```

because we can't turn `@ud` directly into `@dr`, but we can turn
`@ud` into `@` and `@` into `@dr`.

##The smell of size##

Besides these prefixes, which indicate the rendering and/or
meaning of atoms, the odor system has another orthogonal
mechanism to restrict the size of atoms.  Like the prefix, this
mechanism is weak---it is not enforced and trivially evaded.

An odor span contains two parts, both optional: a lowercase
prefix and an uppercase suffix.  The suffix, if present, is a
single character A-Z `c` which indicates an atom of size less
than or equal to `n` bits, where `n` is `1 << (c---'A')`.
Thus, `@tD` is one UTF-8 byte (whatever that means); `@tN`
is a kilobyte or less of UTF-8.

(It's easy to complain from a standards perspective that "UTF-8"
defines a format for bytestreams, not bytes, and at a strict
level it means no more to say "one UTF-8 byte" than to say, say,
"one GIF byte."  But odors are not a strict type system.  It is
possible for a byte to *smell* of UTF-8---or even of GIF.)

When enforcing conversions, `@t` has no size information and can
be used as `@tD`; and `@tD`, of course, can be used as `@t`.  But
using `@tN` as `@tD` is an error.  There is no way to generate
the smell of size from a constant without a cast.  And of course
arithmetic results have no odor at all.

While the utility of this mechanism is debatable, at worst it
serves as a comment which documents the programmer's intentions.

A full table---for convenience, not because you're stupid:

```text
A   1 bit
B   2 bits
C   4 bits
D   1 byte
E   2 bytes
F   4 bytes
G   8 bytes
H   16 bytes
I   32 bytes
J   64 bytes
K   128 bytes
L   256 bytes
M   512 bytes
N   1K
O   2K
P   4K
Q   8K
R   16K
S   32K
T   64K
U   128K
V   256K
W   512K
X   1MB
Y   2MB
Z   4MB
```

You of course can build an atom larger than 4MB, though whether
you should is another question entirely.  But the type system
cannot express a size odor above 4MB.

##Known and unknown odors##

The variety of units and formats which an atom can represent is
essentially infinite.  The set of syntaxes which Hoon can parse
and print is fundamentally limited.

For instance, Hoon has no syntax which means "number of miles."
But within your program, nothing stops you from using the odor
system to distinguish a number of miles from, for instance, a
number of kilometers:

```text
~waclux-tomwyc/try=> `@udm`25.717
25.717
~waclux-tomwyc/try=> `@udk`25.717
25.717
```

The printer has no idea what a `@udm` is, but it knows what a
`@ud` and can print accordingly.  Then, if you have a function
which expects a `@udm` and you try to pass it a `@udk`, it will
fail.  The feature seems banal, but spacecraft have been laid low
by less.

This is clearly a crude mechanism.  If you don't like it, you
don't have to use it.

##Known odors##

Hoon knows about the following odors, with defined meanings:

```text
@c              UTF-32 codepoint
@d              date
  @da           absolute date
  @dr           relative date (i.e., timespan)
@f              yes or no (inverse boolean)
@n              nil
@p              phonemic base
@r              IEEE floating-point
  @rd           double precision  (64 bits)
  @rh           half precision (16 bits)
  @rq           quad precision (128 bits)
  @rs           single precision (32 bits)
@s              signed integer, sign bit low
  @sb           signed binary
  @sd           signed decimal
  @sv           signed base32
  @sw           signed base64
  @sx           signed hexadecimal
@t              UTF-8 text (cord)
  @ta           ASCII text (span)
    @tas        ASCII symbol (term)
@u              unsigned integer
  @ub           unsigned binary
  @ud           unsigned decimal
  @uv           unsigned base32
  @uw           unsigned base64
  @ux           unsigned hexadecimal
```

Each of these forms has a URL-safe syntax, which we'll get to.
Each parses as an atomic constant in Hoon, and each is printed
by the Hoon prettyprinter.

But first, a little more Hoonology.

##The humble type##

What is a Hoon type, anyway?  We know a type is a noun.  So is
everything.  What are the semantics of this noun?

Regardless of what this highly overloaded word may and does and
does mean in every other system of computation that has deployed
it, a type in Hoon has two roles.

One, it defines a set of nouns.  Any finite noun is either in
this set, or not in it.

Two, it ascribes semantics to all nouns in this set.  For
example, a Hoon type exports a semantic namespace.

With this settled, let's start by introducing, purely in an
informal and totally friendly way, the `tile` syntax in which
`type` itself is defined in `/=main=/arvo/hoon/hoon`.  This is
not the full definition of `type`, just a simple subset:

```text
++  type  $|  ?(%noun %void)
          $%  [%atom p=term]
              [%cell p=type q=type]
          ==
```

Again, never mind the syntax.  We can easily describe this subset
of `type` in plain English.

It can be `%noun` (i.e., the atom `1.853.189.998`).  Set:
all nouns.  Examples:

```text
~waclux-tomwyc/try=> :type; *
0
*

~waclux-tomwyc/try=> -:!>(*)
%noun

~waclux-tomwyc/try=> :type; `*`%noun
1.853.189.998
*

~waclux-tomwyc/try=> -:!>(`*`%noun)
%noun
```

It can be the atom `%void`.  Set: no nouns.  We can't show any
examples producing `%void`---by definition, none of them would
terminate.  Because that's what `%void` means.

It can be the cell `[%atom p]`, where `p` is a `term` (`@tas`),
possibly empty (i.e., `0`).  Set: all atoms.  Examples: above.

It can be the triple `[%cell p q]` (i.e., `[%cell [p q]]`), where
each of `p` and `q` is itself a `type`.  Set: all cells of `p`
and `q`.  Examples:

```text
~waclux-tomwyc/try=> :type; [3 4]
[3 4]
[@ud @ud]

~waclux-tomwyc/try=> -:!>([3 4])
[%cell p=[%atom p=%ud] q=[%atom p=%ud]]
```

###The noble cube###

Let's introduce another kind of type here, because we'll need it
to talk about constant syntax:

```text
++  type  $|  ?(%noun %void)
          $%  [%atom p=term]
              [%cell p=type q=type]
              [%cube p=* q=type]
          ==
```

Note that when we enter an ordinary constant, like `42`, its type
`[%atom %ud]` is the set of all atoms (with odor `@ud`, but any
atom can have that or any odor).  Its type is certainly not the
set consisting exclusively of the value `42`.

But here's how we produce this "cubical" constant:

```text
~waclux-tomwyc/try=> :type; %42
%42
%42

~waclux-tomwyc/try=> -:!>(%42)
[%cube p=42 q=[%atom p=%ud]]
```

In general, a `%cube` type contains `p`, a single noun, and `q`,
a base type which provides semantics.

Syntactically, any atomic constant can be preceded by `%` to
generate a cube.  The exception is `@tas`, which always needs `%`
and is always cubical.

##Canonical atom syntaxes##

Let's briefly cover the syntax of each built-in odor.  It would
be counterproductive to specify them exactly here; first, this is
a tutorial rather than a spec, and second the spec is the code.
For the exact semantics, consult `++so` in `hoon.hoon`.  Rather,
we'll explain the form and run through some examples.

If some of these syntaxes seem contrived or odd, bear in mind:
none of them collides with any of the others, and they are all
URL-safe and more.  The canonical atom forms use only lowercase
characters, numbers, `.`, `-`, and `\textasciitilde`.  A cell form adds `_`.

###Unsigned decimal, `@ud`###

Unsigned decimal is the common or neutral atom representation.
It's not very compact and in many cases conveys no intelligible
information at all, but it's impossible to screw up.  `@ud` is
the default print format for both `@u` and `@`---i.e., unsigned
numbers with no printing preference, and opaque atoms.

Hoon's unsigned decimal format is the normal Continental syntax.
It differs from the Anglo-American only in the use of periods,
rather than commas, between groups of 3:

```text
~waclux-tomwyc/try=> 0
0
~waclux-tomwyc/try=> 19
19
~waclux-tomwyc/try=> 1.024
1.024
~waclux-tomwyc/try=> 65.536
65.536
~waclux-tomwyc/try=> (bex 20)
1.048.576
```

An unsigned decimal not broken into groups is a syntax error.
Also, whitespace or even linebreaks can appear between the dot
and the next group.

```text
~waclux-tomwyc/try=> 65.  536
65.536
```

###Unsigned hexadecimal, `@ux`###

`@ux` has the same syntax as `@ud`, except that it's prefixed by
`0x` and uses groups of four.  Hex digits are lowercase only.

```text
~waclux-tomwyc/try=> 0x0
0x0
~waclux-tomwyc/try=> `@ud`0x17
23
~waclux-tomwyc/try=> `@ux`(bex 20)
0x10.0000
~waclux-tomwyc/try=> 0x10.  0000
0x10.0000
```

###Unsigned base64, `@uw`###

###Unsigned base32, `@uv`###

The prefix is `0w` for base64 and `0v` for base32.  The digits
for `@uw` are, in order: `0-9`, `a-z`, `A-Z`, -, \sig:

```text
~waclux-tomwyc/try=> `@ud`0w-
62
```

For `@uv`, the digits are `0-9`, `a-v`.

###Signed decimal, `@sd`###

###Signed hexadecimal, `@sx`###

###Signed base64, `@sw`###

###Signed base32, `@sv`###

###Signed binary, `@sb`###

Obviously, without finite-sized integers, the sign extension
trick does not work.  A signed integer in Hoon is a different way
to use atoms than an unsigned integer; even for positive numbers,
the signed integer cannot equal the unsigned.

The prefix for a negative signed integer is a single `-` before
the unsigned syntax.  The prefix for a *positive* signed integer
is `--`.  The sign bit is the low bit:

```text
~waclux-tomwyc/try=> -1
-1
~waclux-tomwyc/try=> --1
--1
~waclux-tomwyc/try=> `@ud`-1
1
~waclux-tomwyc/try=> `@ud`--1
2
~waclux-tomwyc/try=> `@ud`-2
3
~waclux-tomwyc/try=> `@ud`--2
4
~waclux-tomwyc/try=> `@ux`-0x10
0x1f
~waclux-tomwyc/try=> `@ux`--0x10
0x20
~waclux-tomwyc/try=> `@ud`--0w-
124
~waclux-tomwyc/try=> `@sw`124
--0w-
```

###Absolute date, `@da`###

Urbit dates represent 128-bit chronological time, with 2^64
seconds from the start of the universe to the end.  2^127 is
3:30:08 PM on 226 AD, for reasons not clear or relevant:

```text
~waclux-tomwyc/try=> `@da`(bex 127)
~226.12.5..15.30.08

~waclux-tomwyc/try=> `@da`(dec (bex 127))
~226.12.5..15.30.08

~waclux-tomwyc/try=> `@da`(dec (bex 127))
~226.12.5..15.30.07..ffff.ffff.ffff.ffff
```

The time of day and/or second fragment is optional:

```text
~waclux-tomwyc/try=> `@ux`~2013.12.7
0x8000.000d.2140.7280.0000.0000.0000.0000

~waclux-tomwyc/try=> `@ux`~2013.12.7..15.30.07
0x8000.000d.2141.4c7f.0000.0000.0000.0000

~waclux-tomwyc/try=> `@ux`~2013.12.7..15.30.07..1234
0x8000.000d.2141.4c7f.1234.0000.0000.0000
```

We also do BC:

```text
~waclux-tomwyc/try=> `@ux`~226-.12.5
0x7fff.fffc.afb1.b800.0000.0000.0000.0000
```

The semantics of the time system are that UGT (Urbit Galactic
Time) is GMT/UTC as of leap second 25.  UGT is chronological and
will never add leap seconds, even if UTC continues this mistake.
If a gap appears, it must be resolved in the presentation layer,
with timezones and other human curiosities.

###Relative date, `@dr`###

It's also nice to have a syntax for basic time intervals:

```text
~waclux-tomwyc/try=> `@ux`~s1
0x1.0000.0000.0000.0000

~waclux-tomwyc/try=> `@ux`~m1
0x3c.0000.0000.0000.0000

~waclux-tomwyc/try=> (div ~m1 ~s1)
60

~waclux-tomwyc/try=> (div ~h1 ~m1)
60

~waclux-tomwyc/try=> (div ~h1 ~s1)
3.600

~waclux-tomwyc/try=> (div ~d1 ~h1)
24

~waclux-tomwyc/try=> `@da`(add ~2013.11.30 ~d1)
~2013.12.1
```

There are no `@dr` intervals under a second or over a day.  Since
the resolution is so high, though, `(div \sig s1 1.000.000)` produces
a pretty accurate microsecond.

###Loobean, `@f`###

A loobean, or just `bean`, is 0 or 1.  `0` is yes, `1` is no:

```text
~waclux-tomwyc/try=> `@ud`.y
0
~waclux-tomwyc/try=> `@ud`.n
1
```

People who find this strange are probably strange themselves.

###Nil, `@n`###

Nil indicates an absence of information, as in a list terminator.
The only value is `\sig`, `0`.

```text
~waclux-tomwyc/try=> `@ud`~
0
```

###Unicode text, `@t`###

`@t` is a sequence of UTF-8 bytes, LSB first---sometimes called a
`cord`.  For lowercase numbers and letters, the canonical syntax
is `\sig\sig text`:

```text
~waclux-tomwyc/try=> ~~foo
'foo'
```

Note that the prettyprinter makes an unprincipled exception and
prints the text in a noncanonical format:

```text
~waclux-tomwyc/try=> `@ux`~~foo
0x6f.6f66
```

We want to be able to encode an arbitrary Unicode string as a
single URL-safe token, using no punctuation but `.\sig -`, in `@t`.
Space is `.`, `.` is `\sig.`, `\sig` is `\sig\sig`, `-` is `-`:

```text
~waclux-tomwyc/try=> ~~foo.bar
'foo bar'
~waclux-tomwyc/try=> ~~foo.bar~.baz~~moo-hoo
'foo bar.baz~moo-hoo'
```

For all other ASCII/Unicode characters, insert the Unicode
codepoint in lower-case hexadecimal, followed by `.`.  For
example, for U+2605 "BLACK STAR", write ★ α:

```text
~waclux-tomwyc/try=> ~~foo~2605.bar
'foo★bar'
```

This UTF-32 codepoint is of course converted to UTF-8:

```text
~waclux-tomwyc/try=> `@ux`~~foo~2605.bar
0x72.6162.8598.e26f.6f66
```

###URL-safe ASCII text, `@ta`###

`@ta` encodes the ASCII subset that all canonical atom syntaxes
restrict themselves to.  The prefix is `\sig.`.  There are no escape
sequences except `\sig\sig`, which means `\sig`, and `\sig-`, which means
`\_`.  `-` and `.`\ encode themselves.  No other characters
besides numbers and lowercase letters need apply.

Let's cast these to `@t` to see them quoted:

```text
~waclux-tomwyc/try=> `@t`~.foo
'foo'
~waclux-tomwyc/try=> `@t`~.foo.bar
'foo.bar'
~waclux-tomwyc/try=> `@t`~.foo~~bar
'foo~bar'
~waclux-tomwyc/try=> `@t`~.foo~-bar
'foo_bar'
~waclux-tomwyc/try=> `@t`~.foo-bar
'foo-bar'
```

A `@ta` atom is called a `span`.


###Codepoint, `@c`###

Normally when we build atoms of Unicode text, we use a UTF-8
bytestream, LSB first.  But sometimes it's useful to build atoms
of one or more UTF-32 words.

The codepoint syntax is the same as `@t`, except with a `\sig-`
prefix.  Let's repeat our examples, with hex display:

```text
~waclux-tomwyc/try=> `@ux`~-foo
0x6f.0000.006f.0000.0066

~waclux-tomwyc/try=> `@ux`~-foo.bar
0x72.0000.0061.0000.0062.0000.0020.0000.006f.0000.006f.0000.0066
```

###Phonemic, `@p`###

We've seen `@p` used for ships, of course.  But it's not just for
ships---it's for any short number optimized for memorability, not
for arithmetic.  `@p` is great for checksums, for instance.

That said, `@p` is subtly customized for the sociopolitical
design of Urbit as a digital republic.  For example, one feature
we *don't* want is the ability to see at a glance which carrier
and cruiser issued a destroyer.  Consider the carrier `0x21`:

```text
~waclux-tomwyc/try=> `@p`0x21
~mep
```

It issues `255` cruisers, including `0x4321`:

```text
~waclux-tomwyc/try=> `@p`0x4321
~pasnut
```

Which issues `65.535` destroyers, including `0x8765.4321` and
several successors:

```text
~waclux-tomwyc/try=> `@p`0x8765.4321
~famsyr-dirwes
~waclux-tomwyc/try=> `@p`0x8766.4321
~lidlug-maprec
~waclux-tomwyc/try=> `@p`0x8767.4321
~tidlus-roplen
~waclux-tomwyc/try=> `@p`0x8768.4321
~lisnel-lonbet
```

Of course, anyone who can juggle bits can see that
`\sig famsyr-dirwes` is a close cousin of `\sig lidlug-maprec`.  But she
actually has to juggle bits to do it.  Obfuscation does not
prevent calculated associations, just automatic ones.

But at the yacht level, we actually want to see a uniform 32-bit
space of yachts directly associated with the destroyer:

```text
~waclux-tomwyc/try=> `@p`0x9.8765.4321
~talfes-sibwaclux-tomwyc-famsyr-dirwes
~waclux-tomwyc/try=> `@p`0xba9.8765.4321
~tacbep-ronreg-famsyr-dirwes
~waclux-tomwyc/try=> `@p`0xd.cba9.8765.4321
~bicsub-ritbyt-famsyr-dirwes
~waclux-tomwyc/try=> `@p`0xfed.cba9.8765.4321
~sivrep-hadfeb-famsyr-dirwes
```

###IPv4 address, `@if`###

###IPv6 address, `@is`###

Urbit lives atop IP and would be very foolish to not support
a syntax for the large atoms that are IPv4 and IPv6 addresses.

`@if` is the standard IPv4 syntax, prefixed with `.`:

```text
~waclux-tomwyc/try=> `@ux`.127.0.0.1
0x7f00.0001
```

`@is` is the same as `@if`, but with 8 groups of 4 hex digits:

```text
~waclux-tomwyc/try=> `@ux`.dead.beef.0.cafe.42.babe.dead.beef
0xdead.beef.0000.cafe.0042.babe.dead.beef
```

###IEEE single-precision, `@rs`###

###IEEE double-precision, `@rd`###

###IEEE quad-precision, `@rq`###

###IEEE half-precision, `@rh`###

Hoon does not yet support floating point, so these syntaxes
don't actually work.  But the syntax for a single-precision
float is the normal English syntax, with a `.` prefix:

```text
.6.2832             ::  τ as @rs
.-6.2832            ::  -τ as @
.~6.2832            ::  τ as @rd
.~-6.2832           ::  -τ as @rd
.~~6.2832           ::  τ as @rh
.~~~6.2832          ::  τ as @rq
```

(Hoon is a Tauist language and promotes [International Tau Day](http://tauday.com/).)

###Transparent cell syntax###

By adding `_`, we can encode arbitrary nouns in our safe subset.
The prefix to a canonical cell is `._`; the separator is `_`;
the terminator is `__`.  Thus:

```text
~waclux-tomwyc/try=> ._3_4__
[3 4]

~waclux-tomwyc/try=> :type; ._.127.0.0.1_._0x12_19___~tasfyn-partyv__
[.127.0.0.1 [0x12 19] ~tasfyn-partyv]
[@if [@ux @ud] @p]
```

Those who don't see utility in this strange feature have
perhaps never needed to jam a data structure into a URL.

###Opaque noun syntax###

Speaking of jam, sometimes we really don't care what's inside our
noun.  Then, the syntax to use is a variant of `@uw` prefixed by
`\sig`, which incorporates the built-in `jam` and `cue` marshallers:

```text
~waclux-tomwyc/try=> (jam [3 4])
78.241
~waclux-tomwyc/try=> `@uw`(jam [3 4])
0wj6x
~waclux-tomwyc/try=> (cue 0wj6x)
[3 4]
~waclux-tomwyc/try=> ~0wj6x
[3 4]
```

##Noncanonical syntaxes##

These are syntaxes for constants which don't fit the canonical
character-set constraints.

###Hoon symbol, `@tas`###

`@tas`, a `term`, is our most exclusive odor.  The only
characters permitted are lowercase ASCII, `-` except as the first
or last character, and `0-9` except as the first character.

The syntax for `@tas` is the text itself, always preceded by `%`.
This means a term is always cubical.  You can cast it to `@tas`
if you like, but we just about always want the cube:

```text
~waclux-tomwyc/try=> %dead-fish9
%dead-fish9

~waclux-tomwyc/try=> -:!>(%dead-fish9)
[%cube p=271.101.667.197.767.630.546.276 q=[%atom p=%tas]]
```

The empty `@tas` has a special syntax, `$`:

```text
~waclux-tomwyc/try=> %$
%$
```

A term without `%` is not a constant, but a name:

```text
~waclux-tomwyc/try=> dead-fish9
! -find-limb.dead-fish9
! find-none
! exit
```

###Loobeans, `@f`###

`.y` is a little cumbersome, so we can say `&` and `|`.
The `%` prefix cubes as usual.

```text
~waclux-tomwyc/try=> `@ud`&
0
~waclux-tomwyc/try=> `@ud`|
1
```

###Cords, `@t`###

The canonical `\sig\sig` syntax for `@t`, while it has its place,
is intolerable in a number of ways---especially when it comes to
escaping capitals.  So `@t` is both printed and parsed in a
conventional-looking single-quote syntax:

```text
~waclux-tomwyc/try=> 'foo bar'
'foo bar'
~waclux-tomwyc/try=> `@ux`'foo bar'
0x72.6162.206f.6f66
```

Escape `'` with `\`:

```text
~waclux-tomwyc/try=> 'Foo \'bar'
'Foo \'bar'
~waclux-tomwyc/try=> `@ux`'\''
0x27
```

###Strings###

Text in Hoon is generally manipulated in two ways, depending on
what you're doing: as an atomic cord/span/term, or as a `tape`
which is a list of bytes (_not_ codepoints).

To generate a tape, use double quotes:

```text
~waclux-tomwyc/try=> "foo"
"foo"
~waclux-tomwyc/try=> `*`"foo"
[102 111 111 0]
```

We're getting off the constant reservation, but strings also
interpolate with curly-braces:

```text
~waclux-tomwyc/try=> "hello {(weld "wor" "ld")} is a fun thing to say"
"hello world is a fun thing to say"
```

And they can be joined across space or lines with a `.`:

```text
~waclux-tomwyc/try=> "hello"."world"
"helloworld"
~waclux-tomwyc/try=> "hello". "world"
"helloworld"
```