---
layout: post
title: "Every programmer should care about UI design"
categories: misc
---

I am perpetually shocked by how many programmers think that user interface
design falls firmly in the "not my problem" category. A huge number of
programmers I interview either explicitly say that they dislike UI work or are
completely clueless when it comes to discussing the functionality of a UI.

If you think UIs aren't your problem as a programmer, if you think the only
people who need to worry about them are your UX/UI designers, you are dead
wrong (and I probably won't hire you). Here's why: as a programmer you do UI
design almost every day whether you like it or not.

You disagree? Perhaps you don't realize who the users are that your interface is
for: the other programmers working on your code.

There's a joke that any code that isn't yours is spaghetti code. Part of the
problem is that it's generally harder to read code than it is to write it, but I
think another major reason is that most programmers don't realize that their
code is a UI, and when they don't treat it like a UI they end up with a shitty
one.

Everyone can recognize bad UI when they use it, and working on bad code has
almost identical problems:

* Isn't it frustrating when you have to click through 6 different things to get
  to the one part of the app you actually care about? Isn't it frustrating when
  you have to include a bunch of boilerplate in order to make small functional
  changes to the code?
* Isn't it frustrating when a page is absolutely covered in information, yet the
  one thing you want to find isn't there and it's not clear how to get to it?
  Isn't it frustrating when an object you want to use has 50 public methods,
  but only 3 of them are useful and if you call them in the wrong order your
  program crashes?
* Isn't it frustrating when you input a bunch of info into an app then an error
  occurs and it throws it all away? Isn't it frustrating when your code is super
  permissive when compiling but then throws tons of runtime errors instead?
* Isn't it frustrating when an app labels its UI with meaningless buzzwords like
  "My Solutions", "Insights", etc. making it needlessly difficult to learn how
  to use it? Isn't it frustrating when you have to maintain code with names like
  `EntityOperationManager` and you have absolutely no idea what it is for
  because each of those words has several different meanings and none of them
  are specific?
* Isn't it frustrating when a trivial app takes 10 or 20 seconds to load? Isn't
  it frustrating when it takes 10 minutes for your simple app to build so you
  can test your changes?

The similarities aren't just for people _using_ a UI and _using_ code, they also
apply to developing new code. Any UX/UI designer worth their salt will tell you
that there's only one way to develop a good UI: test it. _Really_ test it. It's
not enough to ask people "How do you think this UI should work?" It's not enough
to show people pictures of a mockup and ask if they like it. No, you need real
users to actually sit down in front of the UI and try to use it.

And yet many software developers I've worked with seem to think that they can
just dream up a software design on a whiteboard and it will be perfect. Or they
use the software design to develop a feature themselves, so it must be a good
software design. But did you test it? Not automated unit tests, did you test the
_UI_ of your software design? Did you show it to other developers who were
unfamiliar with it and see if they could figure it out? Did you consider all of
the different kinds of users who will be interacting with it: the developer
bolting on a brand new feature in 3 months who had no hand in the software
design, the developer working at 3AM to track down a crash that is somehow
related to your changes, the newbie trying to learn the structure of your
code so that they can make good design decisions themselves?

Did you give some thought for those people---your future users---when you were
creating your software UI? Or did you make selfish decisions by making your
software hard to interact with?

## How not to design a better software-UI

Making a software design with a good UI is similar in many ways to designing a
good API for a library or web service: you need to think hard about things like
data formats, stability, understandability, naming; but this is a trap. There
are other aspects to making a good software-UI which are unique, and you ignore
them at your peril.

They key difference is that while an API is often used by complete strangers who
you will never meet, the UI of your software design affects mainly your
teammates. It doesn't need to be nearly as stable, it can (and often must)
evolve as the needs of your team/company/product evolve. It can also rely on
esoteric domain-specific knowledge, or impose harsh restrictions based on your
business needs.

The most frequent mistake I see when developers try to make a good software-UI
is that they start thinking about it as an API. "How would we design this if it
was an open source project used by thousands of people on github?" This often
immediately starts straying into [YAGNI][yagni] territory like, "We can't make
that a hard dependency! What if we need to reuse this component in an
environment where the dependency isn't available?" or "We can't enforce that at
compile time! What if one day we need to reuse this component in a server that
needs to reconfigure its components while running?" Avoid the temptation to
optimize for flexibility above all else: your goal is to deliver value to your
company. Saving on future development cost by having a flexible software design
_is_ one way to do that, but it isn't the only way.

[yagni]: https://martinfowler.com/bliki/Yagni.html

You can deliver much more immediate value by optimizing your design for clarity
of understanding, simplicity of use, early and informative error handling;
basically, all of the parts of the software design related to how other
developers will interact with your code right now, not based on hypothetical
unknown use-cases.

## A better approach to design

Rather than starting my software designs with an application programming
interface (API), I instead like to start developing them with a _programmer
interface_ (PI). In other words, don't try to start by laying out data types and
interfaces, instead start by writing the code as if your software design is
already done. Imagine you are another developer using your finished software
design and try to implement some of your expected use cases.

I call this PI-driven-design, and like test-driven-development it can be a
little hard to wrap your head around at first. Just like how TDD starts by
writing failing tests before the implementation is done, you start your PI
design by writing code that can't compile or run because none of the software
exists yet. The advantage of this is that it immediately strips away a lot of
the complexity that a whiteboard software design introduces. Rather than
creating an explosion of different types and data structures and algorithms and
design patterns, you focus only on the immediate needs of the users of your
software design. Is the code readable? Is the intent of the code obvious by
looking at the names and types? Do the initialized objects do enough to seem
useful or do they seem like they are encapsulating too much functionality?
How will errors manifest if the code is wrong: compiler error, exception,
silent failure? Will the cause of the error be clear to the programmer?

You can still think about flexibility as well, but again focus on how your code
will flex to accommodate other programmers as opposed to specific hypothetical
use cases. Is the code logically organized such that it is easy to find things
later if changes are needed? Is functionality encapsulated in a way that small
functional changes can be implemented easily? One helpful technique for
answering that last question is writing your code such that the confidence of
your assumptions matches with how many changes would be required if the
assumption turns out to be wrong. In other words, try to avoid these two
situations:

### 1. Flexibility über alles

You make your software design extremely flexible and robust, which makes it more
verbose and technically challenging to use. However currently there is only one
known use-case and the flexibility is based only on personal speculation. This
is wasteful. You're spending resources optimizing for a hypothetical situation
while ignoring the immediate extra costs you are incurring by making your
software harder to use. Most likely your software design needs a programmer
interface which hides the flexibility and directly addresses your immediate
needs.

### 2. Leaky assumptions

You simplify your software design by making an assumption that a certain kind of
flexibility won't be needed. However you don't really have evidence supporting
that assumption, and if it turns out to be wrong, lots and lots of code will
need to be refactored. This approach is too risky. If you want to make that
assumption you need to find a way to isolate it to a small part of your software
design such that less extensive refactoring would be required.

### Examples

These problematic situations can arise with any language, any domain, any
application, but here are two common examples to give you a sense of what they
look like in practice.

I've met a lot of programmers who, when adding a property to a class, always add
public getters and setters for it. In most cases I would consider this
user-unfriendly due to overemphasized flexibility. The only "benefit" is that it
provides the maximum amount of flexibility: anything with access to that object
can read it or modify it at any time. But this also makes it much harder for
programmers to use: any code modification around that property might require
refactoring in many other parts of the code, any new interaction with that
property might introduced regressions in other code which was already using it.
A much more user-friendly choice is to do the opposite: make it completely
private. Even better make it private and immutable so that it can only be set
once even inside the instance (if your language supports that). This is
user-friendly because it is the simplest behavior to understand: if you are
working outside of that class you don't have to think about it at all
(encapsulation!), and if you are working inside the class you only have to
consider the value it was initialized with (side-effect-free!). Not every
software design works with such restrictive properties, but you should fight
tooth and nail to give up as little ground as possible. If it must be public, at
least make it immutable. Or if it must be mutable, make it only publicly
readable not writable.

I've also met a lot of programmers who frequently use the singleton pattern.
When I ask why they are using singletons in their design they usually answer, "I
only need one instance." This is a user-unfriendly decision due to leaky
assumptions. The only case where the singleton pattern is truly needed is when
there _must not be_ more than one instance, i.e. there is evidence that having
more than one instance would cause problems in some way. Using a singleton
without this evidence is an assumption which you are forcing on every user of
your software design. In most cases they try to justify this assumption out of
some misguided attempt at making a PI-driven-design, "Look how easy it is to
use: `MySingleton.instance.foobar()`" but as soon you realize you need a second
instance now all of that easy-looking code needs to be refactored and replaced
with dependency injection. Fun! Not to mention how singletons are often just a
thin disguise over what are actually global variables in your code. These are
not user-friendly design decisions, they are lazy-software-designer-friendly!
Most likely the actual problem you are trying to solve is the dependency
injection; a true PI-driven-design would address that directly rather than
taking the easy option of using a singleton.
