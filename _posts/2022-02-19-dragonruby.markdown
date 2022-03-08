---
layout: post
title: "Thoughts on DragonRuby Game Toolkit"
date: 2022-02-19
categories: etc
---

I have always been curious about the [DragonRuby Game Toolkit][drgt] (mainly
because I ❤️ Ruby) but since game dev is only a hobby of mine and there are
already enough free engines out there for me to tinker with, I never bothered to
shell out the $50 for the standard version. Luckily I was recently able to grab
a free copy as part of a game jam promotion, so I've finally been able to give
it a spin. So this is a review/summary of the engine from a hobbyists
perspective.

[drgt]: https://dragonruby.org/toolkit/game

## What is it?

DragonRuby describes itself as "a professional grade 2D game engine", which is
true, but I think too vague. The same could be said of Unity and Godot (ignoring
their optional 3D modes), but DragonRuby could hardly be more different from
them. If you've ever made a game with just C and SDL or just C++ and SFML,
DragonRuby will feel much more familiar to you, because these engines primarily
concern themselves with

* Being cross-platform
* Handling input
* Drawing sprites/text on the screen
* Playing audio files

...and that's basically it. Elements that you might call _very useful_ in game
development (if not essential) such as vector math, animations, physics, a
camera, a scene editor, a UI system etc. are more or less completely absent. In
their place it has all of the precursor elements of these features. For example,
no animations but it understands how to slice up sprite sheets, no vectors but
functions exists for some common geometric calculations, no UI but it has
functions for positioning draw calls relative to the screen dimensions.
Basically you're looking at reinventing these sorts of components if your game
needs them.

There is an unaffiliated open source package system called [Smaug][smaug] but
currently it sits at a measly 23 packages and most of those seem to be related
to [Draco][draco], an implementation of ECS in DragonRuby (which seems a little
odd... why not use Unity/Godot then?).

[smaug]: https://smaug.dev/packages/
[draco]: https://github.com/guitsaru/draco

DragonRuby's philosophy seems to be minimalism. Unlike Unity, which prescribes a
very specific structure for how your game should work, DragonRuby tries hard to
stay out of your way, which I respect. If you want to make a very simple, or
very nontraditional game, this kind of minimalism gives you a nice blank slate
to start from. It also results in some [impressive performance advantages][perf]
over Unity.

[perf]: https://youtu.be/MFR-dvsllA4

All of that makes DragonRuby sound a bit unremarkable, but there is more to it.
Much more.

## DragonRuby's killer features

The main distinction between DragonRuby and SDL/SFML is the language: Ruby. This
isn't just an aesthetic difference, this actually unlocks the amazing, unique
features of DragonRuby.

Probably the biggest feature that DragonRuby has over other engines is that it
has a built-in console. Since Ruby is a dynamic scripting language, this console
isn't restricted to some specific subset of functionality, it uses the very
language that the rest of your game is written in and has access to the exact
same features! The console ends up operating like Unity's editor on steroids:
not only can you examine and tweak the data in your game while it runs, you can
call arbitrary functions, write entirely new functions, and add custom debugging
tools to your game. It's such a huge paradigm shift in game development that I
suspect I have yet to scratch the surface of the possibilities it unlocks. I bet
you could even write an entire game from within the game itself, using only the
console. Not sure why you would, but that shows how powerful this feature is.

Coupled with the console is DragonRuby's hot-reload feature. Whenever you save a
source file, DragonRuby picks up the differences and almost instantly
incorporates them into the running game. This quick feedback makes iteration so
much faster than in other compiler-based engines.

Underlying all of this is DragonRuby's `args`. This object essentially represent
the global state of the entire engine, not only all inputs and outputs, but
_all_ game state (stored in `args.state`). This object persists in the engine as
long as you have it running, independent of the console and hot-reloading. This
is what ties the whole DragonRuby experience together. You can dynamically add a
new object to your game's state in the console, then update your script to
interact with that object, then switch back to the console to perform further
tweaks, etc. It also has built-in serialization support.

And thanks to Ruby's beautiful design, interacting with all of this is fairly
intuitive. `args` more or less operates as a normal Ruby object, with all of the
magic of the engine tucked away under the hood. It's very easy to start simple
and add abstractions as-needed, mixing functional, imperative, and object
oriented styles to suit the structure of your game.

Less essential but still very cool is that it has a built-in demo recorder that
runs in-engine. This makes it easier to test things like asset swaps to see how
they impact your game (without needing to drive yourself crazy playing your game
over and over again).

All of these features put DragonRuby in a interesting position counter to
mainstream engines like Unity: sure it's missing a lot of the basics, but it
also has these amazing fundamental features that would be _extremely_ difficult
to pull off in other engines.

## The bad parts

There are aspects I dislike, however.

The biggest one is that all of the dynamism that I was lauding earlier can be a
double-edged sword. Shared mutable state is probably the #1 cause of
sanity-destroying bugs in my experience and DragonRuby is essentially built
around it. Even when working on a simple game, you might find that the amazingly
fast iteration cycle frequently grinds to a halt when something weird starts
happening and nothing you do seems to fix it... until you restart the engine.
Yep, something you typed in console, something that got hot-reloaded, or some
interaction between the two put something weird into `args` and now it's in
there until you either figure out what it was and undo it or kill the engine.

Second, you might think that an advantage of DragonRuby is that all of the
resources of the Ruby community are at your disposal. But for (totally valid)
technical reasons, DragonRuby actually runs on a fork of [mRuby][mruby] so I
expect that many popular Ruby libraries are actually incompatible with the
engine. I wish this is something Smaug would handle: some kind of vetting system
for finding existing gems which are DragonRuby-compatible.

[mruby]: https://github.com/mruby/mruby

Third, the API design has issues. It provides too many ways to do the same
thing with too little explanation of why. For example you can draw on the screen
using an array, a hash, or an object. From what I've read in the docs, using a
hash is more performant and using an object gives you more control over
rendering order and using an array is for... what? Also how much more performant
is a hash over an object? Should I only introduce objects when absolutely
necessary or is the difference usually not that big? There is basically no
guidance about this stuff, so you just need to try it and see what
happens. Also, despite being in Ruby, the API design is _very_ C-reminiscent.
For example `blendmode_enum: 1` means "alpha blending"; why not `blend_mode:
:alpha`? I'm pretty sure that's nearly identical performance-wise and about 1000
times more readable.

Lastly, and perhaps most annoyingly, the documentation sucks. It's one huge HTML
file with minimal organization. For example, where do you think you go to find
the explanation of how the aforementioned `blendmode_enum` works? "Rendering A
Sprite"? No. "More Sprite Properties As An Array" (subtitle: "Here are all the
properties you can set on a sprite.")? No. How about under
`args.outputs.sprites`? No. It's under "Different Sprite Representations", of
course! It also has gaps. For example, several examples show objects calling the
method `lines` which I cannot find mentioned anywhere outside of source code. By
poking around in console (Ruby FTW) I figured out that it's being included in
`Array` from the `GTK::Primitive::ConversionCapabilities` module but it's still
a mystery to me what it's for.

Also most of the docs are source code dumps of example games. The fact that so
many examples are included is great but it doesn't do much at all in the way of
helping you understand them; you're left to reverse engineer the techniques from
largely undocumented code. It also makes it a huge pain to Ctrl-F in the docs,
since you'll get stuck paging through hundreds of lines of undocumented source
code rather than the single explanation of your search term.

## Conclusion

Overall I actually really like DragonRuby. The documentation issues are
annoying, but the extent of functionality of the engine is so limited that it
isn't possible to get extremely lost in it. And while its flexibility can bite
you, that flexibility does come with huge advantages, and it is also in line
with its theme of minimalism. If you want design your own, more restrictive
framework within DragonRuby, you are free to do so (e.g. Draco with ECS) and it
completely stays out of your way.

If the idea of having a bare-bones engine to build on top of appeals to you and
you don't want to get stuck with the restrictiveness of C/C++, the $50 price tag
is definitely worth it. I'm intrigued by the features you get with higher-tier
licenses but those require an annual subscription and I still haven't made a
dime off of game development.
