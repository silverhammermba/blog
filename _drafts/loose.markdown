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
