---
layout: post
title: "Re: Making Wrong Code Look Wrong"
date: 2020-06-17
categories: etc
---

I just read this fascinating old blog post by Joel Spolsky from 2005, [Making
Wrong Code Look
Wrong](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/).
But I find it fascinating for a completely different reason than you might
expect.

First, to explain the title of his post in a little more detail, the more
"local" you can make bad code, the easier it is to spot, and the better your
code reviews will be. If you have to look in different methods, classes, or
files to figure out if something is bad, you are more likely to miss a problem.

He offers three examples of this in practice:

2. Exceptions make these problems worse because they make every function call
   suspicious. Unless you dig all the way into each function's stack, you have
   no idea if it can interrupt the control flow with an exception.
3. Operator overloading also makes things worse in the same way, by making
   innocuous looking code like `a + b` suspicious since the result can depend on
   the runtime types of its operands.
1. Hungarian notation can actually prevent a lot of these problems as long as
   you use it to denote information that is _not visible in the type alone_.
   E.g. using prefixes `s` and `us` to indicate if a string is "safe" or
   "unsafe". This lets you quickly see where an incorrect conversion is
   occurring.

I 100% agree with all of his observations and his conclusion. But what I find
fascinating is how all of these examples are basically non-issues in modern
languages like Swift.

## Fixing the language

The Swift compiler complains if you throw an exception and don't annotate its
enclosing function with `throws`. I like this solution because it creates a
collaboration between the compiler and the programmer: you might forget about
the possibility of an exception so the compiler forces you to make it more
visible with the annotation. Similarly the compiler complains if you call
something that `throws` and you don't annotate the call with `try`, even if you
are properly handling or propagating the exception. Again, you might forget
which methods are the dangerous ones so the compiler forces you to make it more
visible.

Swift arguably has the same problems with operator overloading that Joel
mentions, but it provides some mitigation by allowing you to define completely
new operators. If you're creating a new type for which `+` has a really
intuitive definition, go ahead and overload it. But if you just want _some kind
of operator_ for your type so you can define a fancy DSL, go ahead and do that
too but just make a new operator! Again it adds that locality by making it
obvious that something weird is going on (though you might have to do some
digging to figure out what that is).

Lastly, in Swift, it's trivial and cheap to make an abstraction like this

{% highlight swift %}
struct UnsafeString {
    let unsafeString: String
}
{% endhighlight %}

This has almost no runtime overhead (it's basically a pointer stored on the
stack) and it easily lets the compiler catch when you tried to implicitly pass
along an unsafe string as a safe string. And in the cases that the compiler
_can't_ catch, it still provides that local readability by forcing you to write
`someVar.unsafeString` whenever you want to actually use it. Even better, you
don't have to clutter up your variable names with Hungarian prefixes.

I don't think Swift is unique in finding language-level solutions to these
problems, it's just what I'm most familiar with right now. From what I've read
about languages like Kotlin and Rust, I suspect they provide similar
functionality.

## The future

What I find most exciting about this is that it is the closest thing we have to
solid evidence that programming language design is actually progressing in a
meaningful sense. Programmers will always argue about which language is better
and it is tempting to fall back on arguments like "Tried and true languages like
Java and C++ will always have a niche!" or "All of these fancy new languages are
just a fad!" I think these examples disprove that.

I'm not claiming that new languages are inherently superior, obviously they can
have their own shortcomings that lead them to fall out of favor. But at the very
least new languages are coming up with practical solutions to serious software
development challenges from the past. 15 years ago Joel Spolsky figured out how
to articulate a general principle of software design, today we have compilers
that understand how to enforce some of those principles.

Maybe Rust won't supersede C, maybe Kotlin won't supersede Java, but it's seems
inevitable that one of their descendants will.
