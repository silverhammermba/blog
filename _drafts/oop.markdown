---
layout: post
title: "Object Oriented Programming is Not Enough"
date: 2021-12-13
categories: etc
---

Suppose you are working with your team on a project in C. You open a `.c` file
and your eyes widen in horror: a dozen global variables, a dozen functions, all
calling each other in a tangled web, exchanging information through entirely
unclear paths. Yes, it's the dreaded "spaghetti code" you've heard about in
school.

Luckily for you, the business is pivoting and the code must be adapted to run on
a toaster, so everything needs to be rewritten in Java; a wonderful opportunity
for refactoring! Easy: those global variables are now _fields_, those functions
are now _methods_, slap the whole thing in a class and you're done! Now you're
using Object Oriented Programming™, which is the industry standard.

What has improved in this situation? You still have a big pile of spaghetti, but
now the pile is in a box with a label on it. It might help a bit. If someone
needs the spaghetti you can now explicitly give them the spaghetti box, which
might make the code slightly easier to understand. Also, if (and this is a _big_
"if") you happen to need a separate copy of that spaghetti it's much easier to
get it.

But I think few good programmers would say that the situation is significantly
better than it was before.

## Power tools

You might say I'm being unfair to OOP, after all no true OOP designer would
write code like that. They would use SOLID and The Clean Architecture to improve
the design and untangle the spaghetti. But this is my point: those design
principles **are not the same as OOP**. They are techniques built on top of OOP
to help you avoid creating boxes full of spaghetti or, even worse, a big pile of
spaghetti made out of boxes. Yuck.

OOP is often defined in terms of its features: abstraction, inheritance,
encapsulation, and polymorphism. But lately, I find this definition more and
more disingenuous. First of all, it cannot be a definition because other
programming paradigms can provide those same features through different
mechanisms. And second, because in a literal sense OOP is just classes, fields,
and methods. Yes, fields can be used for encapsulation, classes can be inherited
and do behave polymorphically, and all of this together allows for the creation
of abstractions. But you can also use all of those same pieces and very easily
produce a huge pile of garbage with spaghetti leaking everywhere.

It's as if the construction industry spent a ton of time talking about
"power-tool-oriented construction" as if there were a single good way to
construct things using power tools. And in school all of the students are shown
the individual power tools: this is the bandsaw, it cuts; this is the drill, it
makes holes; etc. After all, most construction involves cutting and making holes
in things! So with that they are sent out in the world ready to construct
whatever they want, having never learned a single thing about how real, working
things are actually _built_.

Are power tools useful? Yes! Is it important to understand how to use power
tools? Yes! Should you be a bit wary of anyone in construction who says, "I hate
power tools. I avoid them"? Yes! Would you look for new hires in construction
that already have familiarity with power tools? Yes!

But they are. not. enough.
