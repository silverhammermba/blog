---
layout: post
title: "Gradle is a garbage fire"
categories: misc
---

Insanely unstable for something that's been out since 2008!!!!!

You only install gradle to install gradle wrapper, which actually installs
gradle

DSL is fucking awful. You have to learn an entirely separate language (with its
own libraries! that breaks every time you update it!) just to build your actual
code. If you expose this much functionality you better have really fucking
clear errors (e.g. Rust)

Insanely complex, no fucking help at all when it comes to diagnosing build
errors

Waaaay too permissive. Encourages you to make a super custom build for your
special project, which a) makes its interface very hard to understand (not
opinionated enough) and b) does not establish patterns or antipatterns, you're
totally on your own to slowly discover that wisdom through trial and error

Slow as shit, works around it with a daemon (not a real solution), which causes
its own issues (stale env vars). laughably slow compared to other build tools.

Tries way too hard to overgenerlize when its only practical use is for JVM stuff

The only people who use gradle outside of that ecosystem are people who got
stockholm syndrome while working with java and can't be bothered to learn a better
build tool when working outside of Java. No one in their right mind should
choose gradle when you have any other choice.
