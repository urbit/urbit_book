#Preface: Urbit

> \noindent A complex system that works is invariably found to have evolved from a
> simple system that worked. The inverse proposition also appears to be true:
> A complex system designed from scratch never works and cannot be made to
> work. You have to start over, beginning with a working simple system.
> \medskip \\
> Gall's Law


Urbit is a general-purpose programming environment designed to execute a personal cloud computer. We got tired of system software from the 1970s. So we wrote our own. From scratch. Here's what it's made of:

##Nock, a minimal virtual machine

Nock is a homoiconic combinator algebra, not much fancier than SKI combinators. Its spec fits on a T-shirt and gzips to 340 bytes. Nock's data model: a noun is an atom or a cell. An atom is any natural number. A cell is an ordered pair of any two nouns. 

Nock cannot create cycles and needs no garbage collector. Its only arithmetic operator is increment, so it is not inherently efficient; an efficient Nock interpreter works by recognizing standard Nock formulas and matching them to pre-written manual optimizations. We never extend Nock or call out to Unix from it. Nock's goal is extreme commoditization of computing semantics.


##Hoon, a typed functional language 

Hoon is a strict, typed functional language that compiles itself to Nock. The Hoon compiler is 4000 lines of Hoon. Adding standard libraries, the self-compiling kernel is 8000 lines.

Hoon has no particular family relationship to any older language, functional or not. It uses its own type inference algorithm and is as different from Haskell as from Lisp. Its syntax is also completely unfamiliar. In general, Hoon tries to have the same relationship to Nock that C has to assembly - as thin a layer as possible. It is possible to learn Hoon without Nock, but it's probably not a good idea.

As a functional systems language, Hoon is especially good at: metaprogramming, self-virtualization, hotpatching, etc; marshalling and validating untyped data; decoding and encoding binary message formats. Hoon is designed for event programming, so there is no concurrency model.

##Arvo, a functional operating system

Arvo is an event-driven server OS built on the same event library as node.js (libuv). Unlike node.js, Arvo is (a) written in Hoon, (b) isolated from Unix, (c) a persistent single-level store. Arvo can update itself and its data and programs indefinitely from its own state - after the image is created, it lives indefinitely and is never modified externally.

Arvo is modular, but present modules provide a network messaging protocol, a REPL and task manager, a revision-controlled filesystem, a text console, and an HTTP server. The Arvo codebase is about 6500 lines, which is probably too big.

##Clay, a global immutable namespace

Clay is a feature of Arvo - a secure, referentially transparent decentralized filesystem. The root of the namespace is a global PKI - each identity serves its own filesystem tree. Files are nouns, not atoms, with arbitrary tree structure.

Identities, which are also routable addresses, are either key fingerprints or hierarchically assigned short numbers. All numbers are mapped to phonemic strings for memorability. All identities are transferrable and irrevocable; ownership is established by key pinning. The system has a superficial resemblance to Bitcoin, but works more like land than money. It does not use mining or a blockchain.

Since Clay is immutable, all paths contain a revision, which is either a change number, a date, or a label. A request is a special case of a subscription, so syndication is straightforward. Clay also has a synthesis mode which computes a functional namespace on the client side.

##What is Urbit good for?

Urbit is good for everything, of course! It's a new general-purpose computing layer. Once our program of world domination is complete, the old Internet will exist only as an underground series of tubes which transports you to your Urbit.

But more specifically, Urbit is a personal cloud computer. Right now, the cloud computers we use run OSes designed for minicomputers in the '70s. An ordinary user can no more drive a Linux box in the cloud than fly an A320. So she has to sit in coach class as a row in someone else's database. It's definitely air travel. It's not exactly flying.

The user of the future will fly her own computer. She will own and control her own identity and her own data. She will even host her own apps. She will not be part of someone else's Big Data. She will be her own Little Data. Unless she's a really severe geek, she will pay some service to store and execute her Urbit ship - but she can move it anywhere else, anytime, for the cost of the bandwidth.

A user can't manage a general-purpose computer unless she basically understands what it's doing. She may not be a programmer, but she needs at least a rough mental model of her computer's state.

So a personal computer has to be a *simple* computer. This is why we built a new system software stack from scratch, with the goal of bringing it in under 10,000 lines of code. Urbit is about 50% over this complexity budget, but nobody's perfect.
What can it do now?

Urbit at present is not good for anything but screwing around. For screwing around, it answers all your needs and is happy to consume any amount of unwanted time.

It does basically work as described above, though. Urbit at present propagates all its updates through its own filesystem, runs its own chat server over its own network protocol, etc. But as of early 2014, it is an interesting prototype, not a useful device.

##How can I join the revolution?

Just wait. If you're cool enough, someone will send you an invitation.

Psych! No, actually, you can go [here](www.urbit.org/setup).

