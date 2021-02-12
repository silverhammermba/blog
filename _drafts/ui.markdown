---
layout: post
title: "Every programmer should care about UI design"
categories: misc
---

I am perpetually shocked by how many programmers seem to think that user
interface design falls firmly in the "not my problem" category. A huge number of
of programmers I interview either explicitly say that they dislike UI work or
are completely clueless when it comes to discussing the functionality of a UI.

If you think UIs aren't your problem as a programmer, if you think the only
people who need to worry about them are your UX/UI designers, you are dead
wrong (and I probably won't hire you). Here's why: as a programmer you do UI
design every single day whether you like it or not.

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
  you have to include a bunch of boilerplate whenever you want too initiate some
  operation in your code?
* Isn't it frustrating when a page is absolutely covered in information, yet the
  one thing you want to find isn't there and it's not clear how to get to it?
  Isn't it frustrating when an object you want to use has 50 public methods,
  but only 3 of them are useful and if you call them in the wrong order your
  program crashes?
* Isn't it frustrating when you input a bunch of info into an app then an error
  occurs and it throws it all away? Isn't it frustrating when your code is super
  permissive when compiling but then throws tons of runtime errors instead?
* Isn't it frustrating when apps use meaningless buzzwords like "My Solutions",
  "Insights", etc. making it needlessly difficult to know what each button does?
  Isn't it frustrating when you stumble on code using an `EntityOperationManager`
  and you have absolutely no idea what it is for because each of those words has
  several different meanings and none of them are specifc?
* Isn't it frustrating when a trivial app takes 10 or 20 seconds to load? Isn't
  it frustrating when it takes 10 minutes for your simple app to build so you
  can test your changes?

The similarties aren't just for people using a UI and using code, they also
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
related to your changes.
