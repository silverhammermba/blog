---
layout: post
title: "The REAL Ruby Gotchas"
categories: etc
---

most Ruby "gotchas" are just things that Ruby does a little differently from
other languages, but are actually wonderful and consistent once you get used to
the language.

But there are actual gotchas that even seasoned Rubyists will be surprised by.

* Singleton classes of classes have special behavior: they inherit from the
  superclass's singleton. This is hacked in so that "class methods" have
  inheritance
* .. and ... can be used to define flip-flops inside conditionals, there's even
  a warning if you use a range literal
* where do top-level defs end up?
* and/or are actually really useful for control flow. most Ruby devs say to avoid them
* Unary & actually has really low precedence inside method args, because it's enforced at the syntax level
* &nil is a special case, has nothing to do with `to_proc`, needed so you can not pass a block
* TODO
