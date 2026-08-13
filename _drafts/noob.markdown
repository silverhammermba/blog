---
layout: post
title: "Beginner mistakes"
categories: etc
---

Probably want a series of posts about mistakes that junior developers make




Writing doc comments: in a word, what is the thing (class, variable, method)
_for_? That should be the _first_ word in the doc. The rest of the doc should
clarify that. Don't write "This is a method that returns..." or "This variable
stores..." because those are truisms of all methods and variables.




Indenting over multiple lines

    never write code
                    that
                    indents
                    like
                    this

because if you change anything on that first line, you need to re-indent
everything to match and it creates a bunch of noise in the diff



singleton pattern



mutable state, especially related to OOP design and multithreading



misguided single return principle



handling countdowns in the worst possible way. timestamp variables!



testing constants in a redundant and pointless way
