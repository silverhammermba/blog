---
layout: post
title: "Configuring Things or: The Arch Way"
categories: etc
---

How software is configured via a mix of environment, config files, parameters.

How to make things work?

One way is to specify absolutely everything, examples:
* Freeze software version
* Have huge config files
* Specify shitloads of args via aliases
* Create custom scripts to set up environment before each command
* Ultimate version of this is Docker: each command has its own fully-specified
  environment

I think this is largely a delusion: There's no such thing as "I got it working,
now it will work forever". Software almost always changes (bugs, new business
needs, etc.). You either choose to keep up with it (at some pace) or you choose
to be forever behind. The more you (over)specify the harder it is to adapt to
change.

Talk about how *I* configure things, the process of finding the "right way" to
configure, leaving defaults unchanged, similarities with debugging e.g. finding
the minimal config that does what you want, like finding the minimal code that
demonstrates a bug.

Compare this with the Arch Way and why this approach stems naturally from that
philosophy.
