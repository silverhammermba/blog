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
* TODO
