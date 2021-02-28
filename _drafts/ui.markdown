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
* Isn't it frustrating when an app labels its UI with meaningless buzzwords
  buzzwords like "My Solutions", "Insights", etc. making it needlessly difficult
  to learn how to use it? Isn't it frustrating when you have to maintain code
  with names like `EntityOperationManager` and you have absolutely no idea what
  it is for because each of those words has several different meanings and none
  of them are specific?
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
How will errors manifest if the code is wrong: failed build, exception,
silent failure? Will the cause of the error be clear to the programmer?

You can still think about flexibility as well, but again focus on how your code
will flex to accommodate other programmers as opposed to specific hypothetical
use cases. Is the code logically organized such that it is easy to find things
later if changes are needed? Is functionality encapsulated in a way that small
functional changes can be implemented easily? One helpful technique for
answering that last question is writing your code such that the confidence of
your assumptions matches with how many changes would be required if the
assumption turns out to be wrong.

For example, let's say you're making a PI design for a client which is
connecting to a server. Currently your app always connects to one server and it
always uses HTTPS, however you don't know for sure if these assumptions will
always be true. A naïve approach would be to say, "Neither assumption can be
guaranteed at this time, therefore the software must be flexible for either
assumption." So in your PI design you try something like this


TODO: this example kinda sucks. it's too specific. need a more general thing

    // use an object to represent the thing to connect to
    server = ServerFactory.construct(protocol: .https, domain: "foo.bar.com", clientId: myId)
    // a singleton manages a connection pool and opens a new connection or returns the existing one
    connection = ConnectionManager.instance.getConnection(to: server)
    // use the connection
    connection.send(someCommand)

This is not necessarily a _bad_ software design, but I wouldn't call
it programmer-interface-driven. If both assumptions _do_ turn out to be true,
then all of this wonderful flexibility becomes annoying boilerplate. Your
developers will almost certainly start writing their own wrappers to avoid
repeating it all over the place, which is exactly the kind of thing you want to
figure out during your software design process and not later on after it starts
being used. In a way it's just as bad as this naïve design

    // connect to our one server over HTTPS with our one client ID if we haven't yet
    // then use the connection to send the command
    Server.send(someCommand)

This will certainly be easy to use for the moment, but it will also require
significant refactoring throughout the app if either assumption turns out to be
false. Again, the solution here is analyze your confidence in these assumptions
and let that inform your PI design.

Perhaps in this case you know that your security mechanisms are very particular
to TLS and that your commands to the server leverage specific features of HTTP.
If you were to start using a new protocol one day, it would require significant
re-engineering of your whole connection and command design as well. In other
words, you have a high confidence in this assumption--it is already expensive
for your company to break it--therefore it's okay to make your PI design
inflexible in this regard. On the other hand, maybe you've been thinking of
suggesting an admin feature where the app opens multiple different connections
while acting as many clients at once. In other words you already have reason to
doubt that assumption. The key is to balance it, make it easy to get the single
server that you have right now, but also leave the door open to new servers
without much needing to be changed:

    // assume all servers will be similar, so an enum case is sufficient
    connection = ConnectionManager.instance.getConnection(to: .ourCompanyServer, clientId: myId)
    // use it
    connection.send(someCommand)

When it comes to implementing this design, the flexibility could still exist
only at a different layer. The HTTPS assumption is at least partially
encapsulated in the `ConnectionManager` now, so we can still adapt to different
protocols. However this new design forces us to separate that assumption from
the user of the connection: if that expensive assumption change ever occurs, we
won't need to scrutinize every single user of the connection now. Remember that
the PI design alone is note a complete software design, you still have to go
back and fill in a normal software design, but the structure of that design will
now be informed by how we want the code to be used by the developer.

