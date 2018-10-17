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
methods" are actually plain-old instance methods defined in a special class (the
singleton class). In fact you know that there's nothing special at all about
"class methods" because the exact same technique works for _all_ objects.
Classes are just instance of `Class`, after all. Another wonderful non-gotcha of
Ruby.

[array]: https://poignant.guide/dwemthy/

{% highlight ruby %}
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
{% endhighlight %}

You might think that this is certainly weird enough to be a gotcha, but actually
it's quite ordinary. It helps if you think of the singleton class as always
being there, just normally hidden from view because it kinda goes without saying
(_every object_ has a singleton class!). Methods defined in the singleton class
are ordinary methods and they get looked up in the ordinary way, see:

{% highlight ruby %}
foo.singleton_class.superclass
# => Foo
{% endhighlight %}

The class chain really goes `foo.singleton_class < Foo < Object`. That way
`foo.baz` gets looked up in the singleton class before it gets to `Foo`. The
same exact principle works for classes, you have `Foo.singleton_class < Class <
Object`, see:

{% highlight ruby %}
Foo.singleton_class.superclass
# => #<Class:Object>
{% endhighlight %}

**What!?**

That's `Object.singleton_class`. Don't try to puzzle out how this works; as far
as I know this is just hacked into the language. This is the gotcha: a class's
singleton class's superclass is the class's superclass's singleton class. Got
that?

It might be easier to look at an example

{% highlight ruby %}
class A # < Object
end

class B < A
end

B.superclass
# => A
A.superclass
# => Object

B.singleton_class.superclass
# => #<Class:A>
A.singleton_class.superclass
# => #<Class:Object>
{% endhighlight %}

See how the hierarchy of singleton classes matches the hierarchy of normal
classes? Like I said, this is a hack. There's no simple consistent rule that
says this is how singleton classes should work. But it _is_ a nice hack,
because it means that "class methods" work the way you would expect with
inheritance. If you call `B.foo`, this is the search order:

1. `#<Class:B>`
2. `#<Class:A>`
3. `#<Class:Object>`
4. `#<Class:BasicObject>`
5. `Class`

So eventually you get back to `Class`, as logic would dictate. But you take
a little detour first. Luckily this quirk is unlikely to impact your Ruby code.
The surprising part is that it works the way you expect, since doing so involves
a little inheritance magic.

### def

{% highlight ruby %}
def foobar
end
{% endhighlight %}

A method that you can call from anywhere without an explicit receiver. Have you
ever wondered how that works? Hint: just like the previous gotcha, it's another
hack so that the normal method lookup procedure can be used. Personally, I
always expect these to end up in `Kernel` because that's where all of the
leftover methods like `puts`, `rand`, and `loop` are, but that's now how the
Ruby devs chose to do it.

TODO finish this section

### and and/or or

The sort-of-not-really-official [style guide][style] has a strong opinion about
`and` and `or`:

> The `and` and `or` keywords are banned. The minimal added readability is just
> not worth the high probability of introducing subtle bugs.

[style]: https://github.com/rubocop-hq/ruby-style-guide#syntax

I heard this opinion echoed so many times over so many years that I just
accepted it as fact and didn't give it a second thought. But I was a hypocrite!
While shunning `and`/`or` along with the rest of the community, I happily (and
rightly) embraced using `if` and `unless` inline.

{% highlight ruby %}
# Rubyists love this
raise("Failed to save document!") unless document.save

# but they hate this???
document.save or raise("Failed to save document!")
{% endhighlight %}

I don't see anything reprehensible about the second form and would have a hard
time coming up with an objective argument for one of them being _always_
superior.

But what about those "subtle bugs"? The classical example is this

{% highlight ruby %}
x = 3 and 4
# x == 3! Whaaaat?
{% endhighlight %}

But this is just a big misunderstanding. Let's take a step back. Really there
are two totally different concepts of "and" at play here:

* On one hand, there's Boolean conjunction. Like, X is true or false, Y
  is true or false, "X **and** Y" is true if and only if X and Y are both true.
  This is a proper binary operator like + or &times;.
* On the other hand, there's... English. Like "Buy cookies **and** bring them to
  the party" which is just saying to do two things in a certain order (obviously
  you can't bring cookies if you fail to buy any). It's not really math.

Most programming languages, taking after C (probably?) mash these two concepts
together. But **Ruby fixed it**. We've got two syntaxes for the two different
concepts:

{% highlight ruby %}
did_send_invites && house_is_decorated && have_beer
# versus
get_money and buy :cookies and bring_to_party :cookies, :cool_hat
{% endhighlight %}

I think both of these examples are perfectly clear and readable Ruby if you
understand the distinction between `and` and `&&`. Here's a simple test to see
if your code is OK: replace `&&`/`||` with `*` and replace with `and`/`or` with
`if`. If the resulting syntax looks batshit crazy, you're probably using them
wrong.

{% highlight ruby %}
# is this OK?
get_money && buy :cookies && bring_to_party :cookies, :cool_hat
# uhhh, maybe not. need parentheses to make any sense of how this is evaluated
get_money * buy :cookies * bring_to_party :cookies, :cool_hat
# logically this is backwards, but it still reads well, so "and" is a better fit
get_money if buy :cookies if bring_to_party :cookies, :cool_hat

# is this OK?
if did_send_invites and house_is_decorated and have_beer
  # get ready for party
end
# definitely not. dear god no
if did_send_invites if house_is_decorated if have_beer
  # get ready for party
end
# in C this would make more sense (0 is false), but at least it's clear how this
# would be interpreted, so "&&" is a better fit
if did_send_invites * house_is_decorated * have_beer
  # get ready for party
end
{% endhighlight %}


* and/or are actually really useful for control flow. most Ruby devs say to avoid them
* Unary & actually has really low precedence inside method args, because it's enforced at the syntax level
* &nil is a special case, has nothing to do with `to_proc`, needed so you can not pass a block
* TODO

### Honorable Mentions

* super vs super()
* constant lookup is syntactic
* flip-flops, soon to be removed https://bugs.ruby-lang.org/issues/5400
