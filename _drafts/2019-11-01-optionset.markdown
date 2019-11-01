---
layout: post
title: "Swift OptionSets suck"
categories: swift
---

Let's say you're programming some kind of weekly schedule where a user can
toggle days of the week on and off. What type do you use to represent the days
they have selected?

The first obvious choice is to make an `enum` for the days of the week. This is
a textbook enum situation: there is a fixed set of days of the week, and they
are even ordered, so you can meaningfully use their raw values. But how do you
represent the collection of days?

* The easy answer, an array, is no good because then you can have nonsense like
  `[.sunday, .sunday]` or `[.tuesday, .friday, .monday]` which would require
  additional validation. A good rule of thumb: don't pick a type that lets you
  use invalid values!
* A set fixes the first problem with duplicates, and _kinda_ fixes the ordering
  problem (by not having ordering). But it isn't ideal because by losing the
  ordering of the enum values, you have to write extra code to match the set
  contents with the enum raw values if you care about ordering.
* You could use an ordered set but that also only kinda fixes it. You don't have
  duplicates, and you _can_ preserve ordering, but again it's on you to ensure
  you put the elements in the set in the correct order. Also Swift doesn't have
  a native ordered set type so you have to use `NSOrderedSet`. I don't like that
  because in the past it has meant that it will get added one day, and the
  interface will be different. That's annoying to deal with.

Luckily Swift does have the perfect type for us here or, rather, a protocol:
OptionSet. You use it when you have a raw type representing a bit field, then
the protocol provides methods to treat that bit field like a set where you can
add and remove elements or represent it using an array literal.

The _only_ problem here is that OptionSet totally sucks.

## It's not a Sequence

If we make our `DaysOfWeek` OptionSet, we can treat it exactly like a set except
when we can't

{% highlight swift %}
var set: DaysOfWeek = [.sunday, .monday]

set.insert(.wednesday)
set.contains(.wednesday) // true
set.count // Value of type 'DaysOfWeek' has no member 'count'
{% endhighlight %}

It's not that the set can't be counted, Swift just doesn't provide a method for
it. Similarly you can't map or filter it. I can see these being less common
operations for an OptionSet, but they could still be potentially meaningful.
