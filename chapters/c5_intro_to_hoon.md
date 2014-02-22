
Let's learn some Hoon.

Just like we used nock.hoon to write and evaluate Nock programs, we're going to be using a file called app.hoon to write and evaluate Hoon.


If your `$URBIT_HOME` is `urb/`, and your ship is
`~tomsyt-balsen`, the Hoon application template is in

```text
urb/tomsyt-balsen/try/bin/app.hoon
```

or the equivalent, depeding on your own ship name and $URBIT_HOME setting.

If you open `urb/tomsyt-balsen/try/bin/app.hoon` with your preferred text editor you should see something that looks like this:

!:
::  /=main=/bin/app/hoon
::
=>  %=    .
        +
      =>  +
      |%
      ++  word  %hello
      --
    ==
|=  *
|=  ~
^-  bowl
:_  ~  :_  ~
:-  %$
!>
[word "world"]

This is Hoon. Don't worry about what any of this is doing yet. 

You can run this app with:

```text
~tomsyt-balsen/try=> :app
[%hello "world"]
```

In Nock, we found that the most complicated expressions we could practically
write were basic math functions like decrement.  

```text
[ 8                                          ::  push
  [1 0]                                      ::  just 0
  [ 8                                        ::  push
    [ 1                                      ::  quid
      [ 6                                    ::  pick
        [5 [4 0 6] [0 7]]                    ::  same (bump /6) /7
        [0 6]                                ::  /6
        [9 2 [0 2] [4 0 6] [0 7]]            ::  call.2 (cons /2 bump /6 /7)
      ]                                      :: 
    ]                                        :: 
    [9 2 0 1]                                ::  call.2 /1
  ]                                          ::
]                                            :: 

```

If you understand Nock, you can sort of read what decrement is doing, but only if you concentrate pretty hard 


```
++  dec 
  |=  a=@
  =+  b=0
  |-
  ?:  =(a +(b))
    b
  $(b +(b))
```
