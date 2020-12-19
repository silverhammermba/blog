---
layout: post
title: "From Carthage to Cocoapods"
date: 2020-12-19
categories: etc
---

<figure>
<img src="{{ site.baseurl }}/assets/friendshipended_carthage.jpg">
<figcaption>
tl;dr
</figcaption>
</figure>


When I started working on my company's iOS app about three years ago, they had
the awful practice of downloading third-party source code, or frameworks, or
static libraries, hooking 'em up in Xcode, and forgetting about them.
This is obviously a bad idea if you want to be able to audit your third party
dependencies for legal or security reasons, so I made the push to get us using a
dependency manager.

At the time the choice was between Cocoapods and Carthage. Swift Package Manager
was (frankly still is) too immature and little-used by the community and our app
was mostly Objective-C. I chose Carthage.

## Why Carthage?

I knew that Cocoapods was the more popular choice but (partly from my background
in Arch Linux) I've never shied away from supporting the underdog, plus Carthage
aligned better with my software development philosophy.

From a purely technical standpoint, there's nothing wrong with downloading (or
building) a framework and copying it into your project. Lots of third party
libraries even provide instructions about how to do this, since it's basically
the only way to do things if you aren't using a dependency manager. I liked how
Carthage was just a thin wrapper around this basic process, providing the
minimal amount of automation to make the approach less tedious.

Conversely, it felt like Cocoapods took over your app, demanding that you
structure your app in a certain way and performing invasive modifications of
your Xcode project. It wanted you to trust that it was handling things the right
way and to stay out of its way. Again, with my Arch-like philosophy, this rubbed
me the wrong way.

## Signs of trouble

For a few years, Carthage worked pretty well for us. But the list of troubles
kept growing.

With its minimal approach, you really have to know your shit when it comes to
linking libraries in Xcode. Library errors can be extremely opaque and Carthage
will be no help to you whatsoever because it only builds it and copies it into
place. Lots of other Xcode settings can screw things up. I thought this would be
a pro, but after using Carthage for about two years I can say I'd rather hand
off the library management responsibility entirely to the tool so that I can get
on with developing my app.

This was exacerbated with the release of Xcode 12. In true Apple form, Xcode 12
broke everyone's app when it came to running on a simulator (due to libraries
needing explicit x86_64 support to work with non-Apple-silicon simulators). I
eventually found a set of hacks to make things work with Xcode 12 and Carthage,
but they kept breaking with Xcode updates. With Xcode 12.3 I couldn't even get
my Carthage-built libraries to work when building for a real phone, even when
there was no simulator involved at all! Again, I don't want to deal with this
bullshit. Ideally Xcode would handle this stuff, but apparently Apple isn't
interested in that, so having a more invasive tool actually helps here.

Several times we wanted to transition one of our old-unmanaged dependencies to
Carthage or to use a new library with Carthage, but the Carthage build failed in
some way. I think this is a symptom of Carthage being the underdog. The
popularity of Cocoapods really helps here because if the third-party developer
is going to spend time ensuring their library is compatible with a tool, you bet
they are going to test the more popular one first.

Honestly, I wouldn't be surprised if the existence of Cocoapods has dramatically
affected the development of Xcode itself. Because Cocoapods is such a commonly
used tool, as long as it works well, the Xcode dev team has absolutely no
incentive to making Objective-C-compatible library management any easier in
Xcode itself. This hurts the overall Carthage experience, since it relies so
much more on Xcode.

Also Carthage is _slow_. Because a lot of libraries "just work" with Carthage,
that actually means that it needs to download the source code and build those
libraries from scratch every single time. If you're doing a lot of clean builds
(e.g. as part of a continuous integration pipeline), that's a lot of waiting.

Lastly, and frankly this is the least important but it bothers me, is that the
design of the Carthage tool sucks. Part of why I like the Arch philosophy of
using simple tools that don't hold your hand as much is that the tools
themselves can be simple. That is, if you need to understand the inner workings
of the tool, or even modify the tool, it is less daunting. But the Carthage code
is a mess. It is coded _entirely_ in a reactive paradigm. Now don't get me
wrong, reactive programming has its place; it is excellent at handling
asynchronous tasks. But we're talking about a command line tool that you run
occasionally whenever your dependencies change, and when it runs you sit there
and wait for it to finish. The only async work I can think of is downloading
source code, parsing Xcode projects, and running `xcodebuild`. But rather than
using reactive code in only those places that needs async behavior, they decided
to put it literally everywhere. So if you're trying to track down a simple bug
e.g. in how Carthage interprets its command line arguments and matches those up
with the Xcode project before building it, you have to tease apart a bunch of
observable data streams for no good reason. I bet I could write a couple hundred
lines of Ruby code that covers 99% of Carthage's functionality, just by cutting
out the async crap and instead gluing together existing shell tools.

## Switching to Cocoapods

Switching was incredibly easy. I can't believe I didn't to it earlier. The
hardest part was that I wanted to move our source files into a subdirectory (so
that we could have a separate Pods directory outside of the project). This
caused a lot of merge conflicts with other developers and was kind of a
headache, but once it was done the rest was trivial.

One thing that helped even more is that we were already using a few Ruby gems
for our continuous integration process, so all I had to do was add the Cocoapods
gem to our `Gemfile`. Conversely, getting our whole dev team to install carthage
correctly was a bit of a pain.

I wrote a `Podfile`. One immediate benefit here: you can easily see in one place
which parts of your app use which libraries. Ours is broken up into a bunch of
frameworks, and its nice to see which frameworks use third party code as opposed
to Carthage's single list.

Another automatic benefit: the generated `Podfile.lock` shows _recursive_
dependencies, so you can easily see if one of your pods pulled in a bunch of
other crap. Again, Carthage can't do this. Similarly Cocoapods automatically
generates a license file for your third party code. This is something I had to
script manually (and painfully) for Carthage.

The other thing I really like is that, when you install your pods, it warns you
about build settings in your project that could conflict with Cocoapods. This is
exactly the kind of thing I thought I would hate, but I actually love it.
Literally all of the settings it complained about were either old and useless,
incorrect, or could be better set elsewhere (e.g. at the project level rather
than the target level). So as a side effects of using Cocoapods our Xcode
project is now much simpler and has better build settings.
