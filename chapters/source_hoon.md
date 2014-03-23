# Appendix A: hoon.hoon

\begin{codelisting}
\label{code:source_hoon_preface}
\codecaption{}
```
::::::  ::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::  ::::::    Preface                               ::::::
::::::  ::::::::::::::::::::::::::::::::::::::::::::::::::::::
?>  ?=(@ .)                                             ::  atom subject
%.  .                                                   ::  fun with subject
|=  cud=@                                               ::  call it cud
=-  ?:  =(0 cud)                                        ::  if cud is 0
      all                                               ::  then return engine
    (make:all cud)                                      ::  else simple compile
^=  all                                                 ::  assemble engine
  =~                                                    ::  volume stack
%164                                                    ::  version constant
```
\end{codelisting}


```
::::::  ::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::  ::::::    volume 0, version stub                ::::::
::::::  ::::::::::::::::::::::::::::::::::::::::::::::::::::::
~%  %k.164  ~  ~                                        ::
|%                                                      ::
++  stub  164                                           ::  version stub
--                                                      ::
```

This volume describes the current version of Hoon using Urbit's [kelvin versioning system](/blog/2014/02/16/mars/).

At your prompt,

```
    ~tomsyt-balsen/try=> stub
    164
```

```
::::::  ::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::  ::::::    volume 1, Hoon models                 ::::::
::::::  ::::::::::::::::::::::::::::::::::::::::::::::::::::::
~%    %mood
    +
  ~
|%
```

This volume describes various models (such as types and rune functions, for example) used by Urbit. Many of the models are defined in terms of other models. The source itself is listed in alphabetical order for easy reference. The order has been changed here in order of ascending complexity. The easiest to understand models are listed first, although some grouping has been done for related models.


```
++  axis  ,@                                            ::  tree address
```

An `axis` the tree address described by the [Nock
specification](/bestiary/nock/). `,@` means that an `axis` produces an atom.

```
++  bloq  ,@                                            ::  blockclass
```
A bloq is a bare atom most often used in bit manipulation functions for setting
the size of block of bits to operate on. `bloq` should always be read as a
binary power. For example, a `bloq` of 0 means single bits, whereas a `bloq` of
3 means bytes.

```
++  odor  ,@ta                                          ::  atom format
```
An `odor` is an atom format.

`,@ta`, which itself contains the odor `@ta`, means that an odor produces an
atom of ascii text, such as `ta` or 24.948.

```
++  numb  ,@                                            ::  just a number
++  pass  ,@                                            ::  passcode
```

`numb` and `pass` produce atoms. numb is used mainly for stylistic documentation of programmer intent (i.e. "this is just supposed to be number, nothing more") and `pass` is used for passcodes.




###Text

```
++  char  ,@tD                                          ::  UTF-8 atom
```

A char produces a UTF-8 text atom a single byte in length, i.e. a single character.

```
++  tape  (list char)                                   ::  list of characters
```

A tape is a list of chars. This is roughly equivalent to what in many other languages is called a "string". Hoon, however, has two different models that fill the same role that a "string" does in other languages. The other model is a `cord.`

"foo" is a tape.

```
~tomsyt-balsen/try=> `(list ,@)`"foo"
~[102 111 111]
~tomsyt-balsen/try=> `@tD`102
~~f
```

```
++  cord  ,@t                                           :: text atom (UTF-8)
```

A `cord` is an atom of text, or a stream of UTF-8 bits mapped to an atom.

'foo' is a cord.

```
~tomsyt-balsen/try=> `@`'foo'
7.303.014
```

~tomsyt-balsen/try=> `@ub`'foo'
0b110.1111.0110.1111.0110.0110

```
++  span  ,@ta                                          :: text atom (ASCII)

A span is an ASCII text atom. The prefix is `~.`. There are no escape sequences except `~~`, which means `~`, and `~-`, which means `_`. `-` and `.` encode themselves. No other characters besides numbers and lowercase letters are permitted.

~.foo is a span

++  term  ,@tas                                         :: text atom (hoon)

A `term` is a subset of ASCII text. The only characters permitted are lowercase ASCII, `-` except as the first or last character, and 0-9 except as the first character.

`%foo` is a term.

```
++  wall  (list tape)                                   ::  text lines
```

A wall is a list of tapes. Used as text lines.


###Time

++  tarp  ,[d=@ud h=@ud m=@ud s=@ud f=(list ,@ux)]      ::  parsed time

A tarp is a list of atoms representing, in order, day, hour, minute, second and subsecond. The subsecond is itself a list of atoms.

++  date  ,[[a=? y=@ud] m=@ud t=tarp]                   ::  parsed date

A date is a list of a year cell (a loobean head for AD or BC and a year atom), a month atom, and a tarp (a parsed time)

++  time  ,@da                                          ::  galactic time

A time is an atom that maps to a time.

~2014.2.17..14.47.50..0f86 is a time.

~tomsyt-balsen/try=> `@`~2014.2.17..14.47.50..0f86
170.141.184.500.841.997.869.354.824.985.386.942.464
~tomsyt-balsen/try=> `@da`170.141.184.500.841.997.869.354.824.985.386.942.464
~2014.2.17..14.47.50..0f86


###Filesystem

```
++  path  (list span)                                   ::  filesystem location
```

A path is a list of spans used to describe a location in a filesystem.

/~tomsyt-balsen/try is a path

``
~tomsyt-balsen/try=> `(list ,@)`/~tomsyt-balsen/try
~[2.239.102.822.615.623.022.927.944.589.735.038 7.959.156]
~tomsyt-balsen/try=> `(list ,term)`/~tomsyt-balsen/try
~[%~tomsyt-balsen %try]
```

```
++  pint  ,[p=[p=@ q=@] q=[p=@ q=@]]                    ::
```

A pint is a cell of two atom pairs.

```
++  spot  ,[p=path q=pint]                              ::
```
A spot is a cell of a path and a pint. Used for a specific spot in a file

```
++  base  ?([%atom p=odor] %noun %cell %bean %null)     ::
```

A base is any one of the following cases: atom with an odor, noun, cell, loobean, or null.

```
++  axis  ,@                                            ::  tree address
++  also  ,[p=term q=wing r=type]                       ::  alias
++  base  ?([%atom p=odor] %noun %cell %bean %null)     ::
++  beer  $|(@ [~ p=twig])                              ::
++  bloq  ,@                                            ::  blockclass
++  calf  ,[p=(map ,@ud wine) q=wine]                   ::
++  char  ,@tD                                          ::
++  chum  $?  lef=term                                  ::
              [std=term kel=@]                          ::
              [ven=term pro=term kel=@]                 ::
              [ven=term pro=term ver=@ kel=@]           ::
          ==                                            ::
++  claw  $%  [%ash p=twig]                             ::
              [%elm p=twig]                             ::
              [%oak ~]                                  ::
              [%yew p=(map term claw)]                  ::
          ==                                            ::
++  coat  ,[p=path q=vase]                              ::
++  coil  $:  p=?(%gold %iron %lead %zinc)              ::
              q=type                                    ::
              r=[p=?(~ ^) q=(map term foot)]            ::
          ==                                            ::
++  coin  $%  [%$ p=dime]                               ::
              [%blob p=*]                               ::
              [%many p=(list coin)]                     ::
          ==                                            ::
++  cord  ,@t                                           ::
++  date  ,[[a=? y=@ud] m=@ud t=tarp]                   ::  parsed date
++  dime  ,[p=@ta q=@]                                  ::
++  dram  $%  [| p=(map ,@tas dram)]                    ::
              [& p=@ud q=@]                             ::
          ==                                            ::
++  each  |*([a=$+(* *) b=$+(* *)] $%([& p=a] [| p=b])) ::
++  edge  ,[p=hair q=(unit ,[p=* q=nail])]              ::
++  foot  $%  [%ash p=twig]                             ::
              [%elm p=twig]                             ::
              [%oak ~]                                  ::
              [%yew p=(map term foot)]                  ::
          ==                                            ::
++  gear  |*  a=_,*                                     ::  list generator
          $_                                            ::
          =|  b=*                                       ::
          |?                                            ::
          ?@  b                                         ::
            ~                                           ::
          [i=(a -.b) t=^?(..$(b +.b))]                  ::
++  hair  ,[p=@ud q=@ud]                                ::
++  hapt  (list ,@ta)                                   ::
++  like  |*  a=_,*                                     ::  generic edge
          |=  b=_`*`[(hair) ~]                          ::
          :-  p=(hair -.b)                              ::
          ^=  q                                         ::
          ?@  +.b  ~                                    ::
          :-  ~                                         ::
          u=[p=(a +>-.b) q=[p=(hair -.b) q=(tape +.b)]] ::
++  limb  $|(term $%([%& p=axis] [%| p=@ud q=term]))    ::
++  line  ,[p=[%leaf p=odor q=@] q=tile]                ::
++  list  |*  a=_,*                                     ::
          $|(~ [i=a t=(list a)])                        ::
++  metl  ?(%gold %iron %zinc %lead)                    ::
++  odor  ,@ta                                          ::
++  tarp  ,[d=@ud h=@ud m=@ud s=@ud f=(list ,@ux)]      ::  parsed time
++  time  ,@da                                          ::  galactic time
++  tree  |*  a=_,*                                     ::  binary tree
          $|(~ [n=a l=(tree a) r=(tree a)])             ::
++  nail  ,[p=hair q=tape]                              ::
++  numb  ,@                                            ::  just a number
++  pass  ,@                                            ::
++  path  (list span)                                   ::
++  pint  ,[p=[p=@ q=@] q=[p=@ q=@]]                    ::
++  port  $:  p=axis                                    ::
              $=  q                                     ::
              $%  [%& p=type]                           ::
                  [%| p=axis q=(list ,[p=type q=foot])] ::
              ==                                        ::
          ==                                            ::
++  post  $:  p=axis                                    ::
              $=  q                                     ::
              $%  [0 p=type]                            ::
                  [1 p=axis q=(list ,[p=type q=foot])]  ::
                  [2 p=twin q=type]                     ::
              ==                                        ::
          ==                                            ::
++  prop  $:  p=axis                                    ::
              $=  q                                     ::
              [p=?(~ axis) q=(list ,[p=type q=foot])]   ::
          ==                                            ::
++  reef  ,[p=[p=? q=@ud] q=@ud]                        ::
++  ring  ,@                                            ::
++  rule  |=(tub=nail `edge`[p.tub ~ ~ tub])            ::
++  shoe  $%  [%hunk p=tank]                            ::
              [%lose p=term]                            ::
              [%mean p=*]                               ::
              [%spot p=spot]                            ::
          ==                                            ::
++  span  ,@ta                                          ::
++  spot  ,[p=path q=pint]                              ::
++  tank  $%  [%leaf p=tape]                            ::
              [%palm p=[p=tape q=tape r=tape s=tape] q=(list tank)]
              [%rose p=[p=tape q=tape r=tape] q=(list tank)]
          ==                                            ::
++  tape  (list char)                                   ::
++  term  ,@tas                                         ::
++  tiki                                                ::  test case
          $%  [& p=(unit term) q=wing]                  ::  simple wing
              [| p=(unit term) q=twig]                  ::  named wing
          ==                                            ::
++  tile  $&  [p=tile q=tile]                           ::  ordered pair
          $%  [%axil p=base]                            ::  base type
              [%bark p=term q=tile]                     ::  name
              [%bush p=tile q=tile]                     ::  pair/tag
              [%fern p=[i=tile t=(list tile)]]          ::  plain selection
              [%herb p=twig]                            ::  gate
              [%kelp p=[i=line t=(list line)]]          ::  tag selection
              [%leaf p=term q=@]                        ::  constant atom
              [%reed p=tile q=tile]                     ::  atom/cell
              [%weed p=twig]                            ::  example
          ==                                            ::
++  toga                                                ::  face control
          $|  p=term                                    ::  two togas
          $%  [0 ~]                                     ::  no toga
              [1 p=term q=toga]                         ::  deep toga
              [2 p=toga q=toga]                         ::  cell toga
          ==                                            ::
++  twig  $&  [p=twig q=twig]                           ::
          $%                                            ::
            [%$ p=axis]                                 ::
            [%bccb p=tile]                              ::
            [%bccm p=tile]                              ::
            [%bcpt p=wing q=tile]                       ::
            [%bctr p=tile]                              ::
            [%bczp p=base]                              ::
          ::                                            ::
            [%brcb p=tile q=(map term foot)]            ::
            [%brcn p=(map term foot)]                   ::
            [%brdt p=twig]                              ::
            [%brfs p=tile q=(map term foot)]            ::
            [%brkt p=twig q=(map term foot)]            ::
            [%brhp p=twig]                              ::
            [%brls p=tile q=twig]                       ::
            [%brpt p=tile q=tile r=twig]                ::
            [%brtr p=tile q=twig]                       ::
            [%brts p=tile q=twig]                       ::
            [%brwt p=twig]                              ::
          ::                                            ::
            [%clcb p=twig q=twig]                       ::
            [%clcn p=tusk]                              ::
            [%clfs p=twig]                              ::
            [%clkt p=twig q=twig r=twig s=twig]         ::
            [%clhp p=twig q=twig]                       ::
            [%clls p=twig q=twig r=twig]                ::
            [%clsg p=tusk]                              ::
            [%cltr p=tusk]                              ::
            [%clzz p=tusk]                              ::
          ::                                            ::
            [%cncb p=wing q=tram]                       ::
            [%cncl p=twig q=twig]                       ::
            [%cndt p=twig q=twig]                       ::
            [%cnhp p=twig q=tusk]                       ::
            [%cntr p=wing q=twig r=tram]                ::
            [%cnkt p=twig q=twig r=twig s=twig]         ::
            [%cnls p=twig q=twig r=twig]                ::
            [%cnsg p=wing q=twig r=twig]                ::
            [%cnts p=wing q=tram]                       ::
            [%cnzy p=term]                              ::
            [%cnzz p=wing]                              ::
          ::                                            ::
            [%dtkt p=twig]                              ::
            [%dtls p=twig]                              ::
            [%dtzy p=term q=@]                          ::
            [%dtzz p=term q=*]                          ::
            [%dttr p=twig q=twig]                       ::
            [%dtts p=twig q=twig]                       ::
            [%dtwt p=twig]                              ::
          ::                                            ::
            [%hxgl p=tusk]                              ::
            [%hxgr p=tusk]                              ::
          ::                                            ::
            [%ktbr p=twig]                              ::
            [%ktdt p=twig q=twig]                       ::
            [%ktls p=twig q=twig]                       ::
            [%kthp p=tile q=twig]                       ::
            [%ktpm p=twig]                              ::
            [%ktsg p=twig]                              ::
            [%ktts p=toga q=twig]                       ::
            [%ktwt p=twig]                              ::
          ::                                            ::
            [%sgbr p=twig q=twig]                       ::
            [%sgcb p=twig q=twig]                       ::
            [%sgcn p=chum q=twig r=tyre s=twig]         ::
            [%sgfs p=chum q=twig]                       ::
            [%sggl p=$|(term [p=term q=twig]) q=twig]   ::
            [%sggr p=$|(term [p=term q=twig]) q=twig]   ::
            [%sgbc p=term q=twig]                       ::
            [%sgls p=@ q=twig]                          ::
            [%sgpm p=@ud q=twig r=twig]                 ::
            [%sgts p=twig q=twig]                       ::
            [%sgwt p=@ud q=twig r=twig s=twig]          ::
            [%sgzp p=twig q=twig]                       ::
          ::                                            ::
            [%smcl p=twig q=tusk]                       ::
            [%smdt p=twig q=tusk]                       ::
            [%smdq p=(list beer)]                       ::
            [%smsg p=twig q=tusk]                       ::
            [%smsm p=twig q=twig]                       ::
          ::                                            ::
            [%tsbr p=tile q=twig]                       ::
            [%tscl p=tram q=twig]                       ::
            [%tscn p=twig q=twig]                       ::
            [%tsdt p=wing q=twig r=twig]                ::
            [%tsfs p=twig q=twig]                       ::
            [%tsgl p=twig q=twig]                       ::
            [%tshp p=twig q=twig]                       ::
            [%tsgr p=twig q=twig]                       ::
            [%tskt p=twig q=twig r=twig s=twig]         ::
            [%tsls p=twig q=twig]                       ::
            [%tspm p=tile q=twig]                       ::
            [%tspt p=tile q=twig]                       ::
            [%tstr p=term q=wing r=twig]                ::
            [%tssg p=tusk]                              ::
          ::                                            ::
            [%wtbr p=tusk]                              ::
            [%wthp p=wing q=tine]                       ::
            [%wthz p=tiki q=tine]                       ::
            [%wtcl p=twig q=twig r=twig]                ::
            [%wtdt p=twig q=twig r=twig]                ::
            [%wtkt p=wing q=twig r=twig]                ::
            [%wtkz p=tiki q=twig r=twig]                ::
            [%wtgl p=twig q=twig]                       ::
            [%wtgr p=twig q=twig]                       ::
            [%wtls p=wing q=twig r=tine]                ::
            [%wtlz p=tiki q=twig r=tine]                ::
            [%wtpm p=tusk]                              ::
            [%wtpt p=wing q=twig r=twig]                ::
            [%wtpz p=tiki q=twig r=twig]                ::
            [%wtsg p=wing q=twig r=twig]                ::
            [%wtsz p=tiki q=twig r=twig]                ::
            [%wtts p=tile q=wing]                       ::
            [%wtzp p=twig]                              ::
          ::                                            ::
            [%zpcb p=spot q=twig]                       ::
            [%zpcm p=twig q=twig]                       ::
            [%zpcn ~]                                   ::
            [%zpfs p=twig]                              ::
            [%zpgr p=twig]                              ::
            [%zpsm p=twig q=twig]                       ::
            [%zpts p=twig]                              ::
            [%zpwt p=$|(p=@ [p=@ q=@]) q=twig]          ::
            [%zpzp ~]                                   ::
          ==                                            ::
++  tine  (list ,[p=tile q=twig])                       ::
++  tusk  (list twig)                                   ::
++  tyre  (list ,[p=term q=twig])                       ::
++  tyke  (list (unit twig))                            ::
++  tram  (list ,[p=wing q=twig])                       ::
++  tone  $%  [%0 p=*]                                  ::
              [%1 p=(list)]                             ::
              [%2 p=(list ,[@ta *])]                    ::
          ==                                            ::
++  nock  $&  [p=nock q=nock]                           ::
          $%  [%0 p=@]                                  ::
              [%1 p=*]                                  ::
              [%2 p=nock q=nock]                        ::
              [%3 p=nock]                               ::
              [%4 p=nock]                               ::
              [%5 p=nock q=nock]                        ::
              [%6 p=nock q=nock r=nock]                 ::
              [%7 p=nock q=nock]                        ::
              [%8 p=nock q=nock]                        ::
              [%9 p=@ q=nock]                           ::
              [%10 p=?(@ [p=@ q=nock]) q=nock]          ::
              [%11 p=nock]                              ::
          ==                                            ::
++  toon  $%  [%0 p=*]                                  ::
              [%1 p=(list)]                             ::
              [%2 p=(list tank)]                        ::
          ==                                            ::
++  tune  $%  [%0 p=vase]                               ::
              [%1 p=(list)]                             ::
              [%2 p=(list ,[@ta *])]                    ::
          ==                                            ::
++  twin  ,[p=term q=wing r=axis s=type]                ::
++  type  $|  ?(%noun %void)                            ::
          $%  [%atom p=term]                            ::
              [%bull p=twin q=type]                     ::
              [%cell p=type q=type]                     ::
              [%core p=type q=coil]                     ::
              [%cube p=* q=type]                        ::
              [%face p=term q=type]                     ::
              [%fork p=type q=type]                     ::
              [%hold p=(list ,[p=type q=twig])]         ::
          ==                                            ::
++  typo  type                                          ::  old type
++  udal                                                ::  atomic change (%b)
          $:  p=@ud                                     ::  blockwidth
              q=(list ,[p=@ud q=(unit ,[p=@ q=@])])     ::  indels
          ==                                            ::
++  udon                                                ::  abstract delta
          $:  p=umph                                    ::  preprocessor
              $=  q                                     ::  patch
              $%  [%a p=ulna]                           ::  trivial replace
                  [%b p=udal]                           ::  atomic indel
                  [%c p=(urge)]                         ::  list indel
                  [%d p=upas q=upas]                    ::  tree edit
              ==                                        ::
          ==                                            ::
++  ulna  ,[p=* q=*]                                    ::  from to
++  umph                                                ::  change filter
          $|  $?  %a                                    ::  no filter
                  %b                                    ::  jamfile
                  %c                                    ::  LF text
              ==                                        ::
          $%  [%d p=@ud]                                ::  blocklist
          ==                                            ::
++  unce  |*  a=_,*                                     ::  change part
          $%([%& p=@ud] [%| p=(list a) q=(list a)])     ::
++  unit  |*  a=_,*                                     ::  maybe
          $|(~ [~ u=a])                                 ::
++  upas                                                ::  tree change (%d)
          $&  [p=upas q=upas]                           ::  cell
          $%  [%0 p=axis]                               ::  copy old
              [%1 p=*]                                  ::  insert new
              [%2 p=axis q=udon]                        ::  mutate!
          ==                                            ::
++  urge  |*(a=_,* (list (unce a)))                     ::  list change
++  vase  ,[p=type q=*]                                 ::  type-value pair
++  vise  ,[p=typo q=*]                                 ::  old vase
++  wall  (list tape)                                   ::  text lines
++  wing  (list limb)                                   ::
++  wine  $|  ?(%noun %path %tank %void %wall %wool %yarn)
          $%  [%atom p=term]                            ::
              [%core p=(list ,@ta) q=wine]              ::
              [%face p=term q=wine]                     ::
              [%list p=term q=wine]                     ::
              [%pear p=term q=@]                        ::
              [%pick p=(list wine)]                     ::
              [%plot p=(list wine)]                     ::
              [%stop p=@ud]                             ::
              [%tree p=term q=wine]                     ::
              [%unit p=term q=wine]                     ::
          ==                                            ::
++  woof  (list $|(@ud [p=@ud q=@ud]))                  ::  udon transform
++  wonk  |*(veq=edge ?@(q.veq !! p.u.q.veq))           ::
::                                                      ::
++  map  |*  [a=_,* b=_,*]                              ::  associative array
         $|(~ [n=[p=a q=b] l=(map a b) r=(map a b)])    ::
++  qeu  |*  a=_,*                                      ::
         $|(~ [n=a l=(qeu a) r=(qeu a)])                ::
++  set  |*  a=_,*                                      ::
         $|(~ [n=a l=(set a) r=(set a)])                ::
--                                                      ::
```
