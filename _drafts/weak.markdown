---
layout: post
title: "weak is the worst part of Swift"
date: 2022-04-04
categories: etc
---

Worst part of Swift: object A wants to call a method on object B to do some
async work. B reports back via delegate/callback. Immediately two questions:

1. What happens if A drops its reference to B?
2. What happens if whatever was holding A drops its reference to A?

The difference in behavior and also whether or not you have a bug in your code
often depends on a single `weak` keyword, which can appear in any number of
places. It can also be recursive: the behavior of your program can depend on
what B is doing, because B may asynchronously call out to some object C and you
have to answer the same set of questions again.

A change in B or C can cause a change in the behavior of A such that it's
virtually impossible to encapsulate the liftetime of an object in Swift.

How does async/await relate to this?
