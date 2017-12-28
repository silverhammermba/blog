---
layout: post
title: "Loose coupling as an anti-pattern"
categories: etc
---

Article about how loose coupling in OOP can actually make the design worse by
hiding code dependencies.

I'm having a hard time thinking of  a really good example. I think this happens
when you're already violating OOP principles and rather than fixing them
properly you throw in some loose coupling or something that looks like
dependency injection.

The general idea I have is that tight coupling and non-injected dependencies are
often bad, but there are ways to get rid of them that don't actually result in
loose coupling or DI.

OR

Maybe take the angle of "dependency injection" being a conflation of three things:

1. An object being passed its dependencies instead of getting them itself (abuse
   of service locator)
2. Depending only on interfaces, every class having an associated interface
3. Using a framework to automatically resolve interface dependencies

1 is "pure" DI, just good advice on how to structure an OO app.

2 is a nice-sounding idea, but reeks of YAGNI. Implicit is that the interface
will actually be well-designed, but with only one concrete class it is likely to
need extensive changes if ever you construct an alternative. The most practical
use-case is mocking unit tests, but this should be a special case not
ubiquitous.

3 is mainly a solution to the problem invented by 2: way too many interfaces. It
is appealing in that it provides a centralized location for the dependency
graph, but is that really needed? It breaks encapsulation and it encourages
holistic rather than localized designs.

I think this ties into my original point that you can take a tightly coupled
design, apply points 1-3, and end up with something that _looks_ loosely coupled
but is really just the same as before, just harder to read.
