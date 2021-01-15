---
layout: post
title: "Every programmer should care about UI design"
categories: misc
---

I am perpetually shocked by how many programmers seem to think that user
interface design falls firmly under "not my problem". I'd estimate that a
majority of programmers I interview either explicitly say that they dislike UI
work or are completely clueless when it comes to discussing the functionality of
a UI.

If you think UIs aren't your problem as a programmer, if you think the only
people who need to worry about them are your UX/UI designers, you are dead
wrong (and I probably won't hire you). Here's why: as a programmer you do UI
design every single day whether you like it or not.

You disagree? Perhaps you don't realize who the users are that your interface is
for: the other programmers working on your code.

There's a joke that any code that isn't yours is spaghetti code. Half of the
reason for this is because it's harder to read code than it is to write it, but
I'd argue that the other half is because most programmers don't realize that
their code is a UI, and because they don't treat it like a UI they end up with a
shitty one.

TODO: compare stuff like this:

waiting 10 seconds for an app to load: waiting an hour for your trivial app to
build

6 clicks needed to get to the one page of a site that you care about:
instantiating three objects and passing 12 parameters to perform some trivial
operation

a website being covered in information, but none of it is what you need and it
isn't clear where the thing you want is: a class with 30 public methods, but
it's only safe to call a few of those methods and you better do it in the right
order or things will crash

TODO: talk about how unrealistic it is to expect a UI to be good without
testing, yet many developers make a software design before testing it out on
other developers by making a POC
