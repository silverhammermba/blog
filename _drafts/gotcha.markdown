---
layout: post
title: "The REAL Ruby Gotchas"
categories: etc
---

Most Ruby "gotchas" are just things that Ruby does a little differently from
other languages, but are actually wonderful and consistent once you get used to
the language. If you Google "Ruby gotchas", none of the top-five results mention
what I would consider a _real_ gotcha. They're clearly written from the
perspective of someone just learning Ruby (or just learning programming!) and
would be things that I would expect any experienced Ruby programmer to know.

> [T]hey come up to me and say, "I was surprised by this feature of the
> language, so therefore Ruby violates the principle of least surprise." Wait.
> Wait. The principle of least surprise is not for you only. The principle of
> least surprise means principle of least _my_ surprise [sic]. And it means the
> principle of least surprise after you learn Ruby very well.
> --[Matz][matz]

[matz]: https://www.artima.com/intv/ruby4.html

Matz designed Ruby in a way that maximizes its internal consistency, I think
because he knew how much these little surprises and uncertainty about a language
impact your productivity. Once you learn Ruby's rules---which _can_ be
surprising from the perspective of other languages---they're generally very easy
to remember because they have a certain intuitive elegance to them.

But not always.

## The Gotchas

In my opinion, these are truly surprising facts about Ruby. Things that---even
after years of using Ruby---sent me packing to Google and led to a couple hours
of experimentation to try to understand some quirk of the language.

### Singleton superclass

If you've plumbed the depths of [Dwemthy's Array][array], you know that "class
methods" are just plain-old instance methods. The only special bit is that these
methods are defined in a _secret class_ that is dynamically created after the
fact as your new class so that it can intercept method calls before they reach
your normal class.

[array]: https://poignant.guide/dwemthy/

In fact you know that "class methods" don't exist, because this exact same
technique can be used on classes and instances alike because at the end of the
day everything's just an object and who cares if that object is an instance of
`Class` or something else? Another wonderful non-gotcha of Ruby.

    class Foo
      def self.bar
      end
    end

    Foo.singleton_class.instance_methods false
    # => [:bar]
    Foo.is_a? Class
    # => true
    Foo.is_a? Foo.singleton_class
    # => true, our secret parent!

    # same thing works on instances

    foo = Foo.new

    def foo.baz
    end

    foo.singleton_class.instance_methods false
    # => [:baz]
    foo.is_a? Foo
    # => true
    foo.is_a? foo.singleton_class
    # => true

You might think that this is certainly weird enough to be a gotcha, but actually
it's quite ordinary. It helps if you think of the singleton class as always
being there, just normally hidden from view because it kinda goes without saying
(_every object_ has a singleton class!). Methods defined in the singleton class
are ordinary methods and they get looked up in the ordinary way, see:

    foo.singleton_class.superclass
    # => Foo

The class chain really goes `foo.singleton_class < Foo < Object`. That way
`foo.baz` gets looked up in the singleton class before it gets to `Foo`. The
same exact principle works for classes, you have `Foo.singleton_class < Class <
Object`, see:

    Foo.singleton_class.superclass
    # => #<Class:Object>

**What!?**

That's `Object.singleton_class`. Don't even try to puzzle out how this works; as
far as I know this is just hacked into the language. And this is the gotcha: a
class's singleton class's superclass is its superclass's singleton class. Got
that?


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

### Honorable Mentions

* super vs super()
* constant lookup is syntactic
