# Introductory Hoon

Let's learn some Hoon.


## What is Hoon?

[how Hoon compiles to Nock]


Just like we used nock.hoon to write and evaluate Nock programs, we're going to be using a file called app.hoon to write and evaluate Hoon.

[quick and dirty tutorial on how to evaluate Hoon in Arvo]


## Glyphs and Runes

ace  space      gal  <          per  )
bar  |          gar  >          sel  [
bas  \          hax  #          sem  ;
buc  $          hep  -          ser  ]
cab  _          kel  {          sig  ~
cen  %          ker  }          soq  '
col  :          ket  ^          tar  *
com  ,          lus  +          tec  `
doq  "          pam  &          tis  =
dot  .          pat  @          wut  ?
fas  /          pel  (          zap  !

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::                section 2eF, parsing (ascii)          ::
::
++  ace  (just ' ')                         :: space, ace
++  gal  (just '<')                         :: Greater Left
++  gar  (just '>')                         :: Greater Right
++  pel  (just '(')                         :: Paren Left
++  per  (just ')')                         :: Paren Right
++  kel  (just '{')                         :: BracKet Left
++  ker  (just '}')                         :: Bracket Right, 
++  sel  (just '[')                         :: Cell Left 
++  ser  (just ']')                         :: Cell Right
++  doq  (just '"')                         :: Double Quote
++  soq  (just '\'')                        :: Single Quote
++  col  (just ':')                         :: Colon
++  sem  (just ';')                         :: Semicolon
++  com  (just ',')                         :: Comma
++  dot  (just '.')                         :: Period (looks like a dot)
++  tec  (just '`')                         :: Backtick (tick sounds like tec)
++  sig  (just '~')                         :: a squiggle used in a signature
++  zap  (just '!')                         :: Onomatopoeia. zap! bang!
++  pat  (just '@')                         :: At sign 
++  hax  (just '#')                         :: Hash tag
++  buc  (just '$')                         :: Dollar sign, buck
++  cen  (just '%')                         :: Percent
++  ket  (just '^')                         :: Caret 
++  pam  (just '&')                         :: Ampersand, pampersand 
++  tar  (just '*')                         :: star, tar
++  hep  (just '-')                         :: Hyphen 
++  cab  (just '_')                         :: Caboose
++  lus  (just '+')                         :: plus, lus
++  tis  (just '=')                         :: it is, 'tis 
++  bar  (just '|')                         :: vertical bar
++  bas  (just '\\')                        :: backslash
++  fas  (just '/')                         :: forward slash
++  wut  (just '?')                         :: misspelling of "what?"


## Basic Runes

Nock Runes:
.?, cell test, Nock 3
.+, +, increment, Nock 4
.=(a b), =(a b), equality test, Nock 5
.*(a b), evaluate Nock code, with subject a and formula b

~>(a b), pass a hint, Nock 10

?:(a b c), if a, then b, else c, Nock 6
=>(a b), use a as subject of b, Nock 7
=+(a b), push a onto subject , Nock 8

!!, crash, (use the formula [0 0], which always crashes)


## Basic Types: Axils and Atom Odors

Axils

@ atom
^ cell
* noun
? loobean
~ null


Odors

@c              UTF-32 codepoint
@d              date
  @da           absolute date
  @dr           relative date (ie, timespan)
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

@if 
@is

Size odors

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

## Cores and Arms

[refresher on Cores at nock level, discussion of arms]

bartis, barhep, centis, buc

## Basic Math Functions

[explain dec, add, mul ...]

## Noun Surgery and Bit Logic

bloqs


## Noun Orders



## Time

## Lists

bartar,

snag, swag, slag, scag etc.

## Signed Ints

++  si

## Modulo bloq

++  fe

## Phonetic Base



## Unit

## Containers
