---
layout: post
title: "Easy Animations in DragonRuby with Enumerators"
categories: etc
---

I've always struggled with doing programmatic animations in game engines, and
DragonRuby is no exception. The result is always very math heavy, easy to screw
up, and hard to understand even after you get it working. Say you want to
animate a sprite moving back and forth with the following parts to the
animation:

1. keep the sprite stationary on the left for 0.5 seconds
2. over the next 1 second, smoothly move the sprite to the right
3. keep the sprite stationary on the right for 0.5 seconds
4. over the next 1 second, smoothly move the sprite to the left
5. repeat

Doing that using the built-in animation functions in DragonRuby involves adding
code like this to your `tick` method:

```ruby
class Numeric
  # linear interpolate from start to finish as this number varies from 0 to 1
  def lerp start, finish
    ((finish - start) * self + start).to_f
  end
end

# determine total animation length ahead of time
total_duration = 3.seconds
# mod the tick count so animation will repeat
t = args.state.tick_count % total_duration

# first, guess that we're in the left-to-right part
progress = args.easing.ease(
  0.5.seconds, # delay before this part of the animation
  t,           # tick count (modified, since we're looping)
  1.seconds,   # duration of this part of animation
  easing_func  # a proc to smooth the animation (more on this later)
)
args.state.sprite_pos = progress.lerp(320, 960)
# if that part is done, we must be in the right-to-left part
if progress >= 1
  progress = args.easing.ease(
    2.seconds,  # delay before animation, taking into account the previous parts
    t,          # remaining args same as before
    1.seconds,
    easing_func
  )
  args.state.sprite_pos = progress.lerp(960, 320)
end
```

This sucks.

If you've never struggled with this stuff before, I'll spell it out. It sucks
that we have to calculate the overall animation duration ahead of time; it sucks
that we have to calculate a cumulative delay for each additional part of the
animation; it sucks that we need an explicit mechanism for determining which
part of the animation we should be in; it sucks that each frame has to
re-compute parts of the animation that aren't active; it sucks that this code
will get harder and harder to reason about as we add more phases to the
animation; and above all it sucks that our original human-readable description
of the animation is now completely hidden in the code. Good luck reading this
tomorrow and understanding how it effects the desired behavior.

But two months ago, I stumbled upon a [blog post][coroutines] about how to solve
this sort of problem in Unity. I was curious if there's an analogus way to apply
the technique to DragonRuby and I finally got the time to figure it out.

[coroutines]: https://michaelbitzos.com/devblog/programmatic-animation-using-coroutines

## Enumerator

First, a brief diversion into a very cool part of Ruby: enumerators. In Ruby we
have simple loops:

```ruby
i = 0
loop do
  puts i
  i = 3 * i + 4
  break if i > 100
end
# outputs 0, 4, 16, 52
```

But loops are all-or-nothing: once you start running them you have to wait for
them to finish... or do you? Enter `Enumerator`, with this you get fine-grained
lazy evaluation:

```ruby
i = 0
e = Enumerator.new do |yielder|
  loop do
    yielder << i
    i = 3 * i + 4
    break if i > 100
  end
end
e.next # 0
e.next # 4
e.next # 16
e.next # 52
e.next # raises StopIteration
```

This is magic! It looks misleadingly simple because the Ruby interpreter is
doing some cool stuff under the hood for us. That `yielder << i` line is the
real key: that's where the interpreter suspends execution of the block and saves
it to memory. The enumerator remains there, dormant, until we call `next` and it
briefly springs to life until the next `yielder << i` line is hit. The other key
part is that the enumerator is constructed with a block so, like all blocks, it
captures its local scope. That means the `i` variable remains accessible to it.
That would be the case even if we were to return the enumerator from this scope:
the local variable would be gone but the enumerator asleep on the heap would
still have a reference to it until it is done enumerating.

This is exactly what we need to solve our animation problem.

## eease

A simple tool:

```ruby
def ecount duration, &blk
  enum = (0...duration).each
  return enum unless blk
  enum.lazy.map &blk
end

ecount(30) # construct an enumerator that yields 0..29
ecount(2.seconds) # construct an enumerator that yields 0..119
```

In DragonRuby, `x.seconds == x * 60` since it runs at a constant 60 FPS.
Maybe you see where this is going?

```ruby
# in tick
args.state.animation ||= ecount(2.seconds) { |i|
  puts "Animation is on frame #{i}"
}

begin
  args.state.animation.next
rescue StopIteration
  # animation is done
end
```

By calling `next` each frame, this enumerator now represents a real-time
animation (currently just "animating" some text into the console, but you get
the idea)! Already this solves a lot of our problems:

* We no longer need to externally manage the duration of the animation. It
  remembers what duration it was initialized with and it tells us when it is
  done (via `StopIteration`)
* This doesn't require precomputing an absolute start-time based on the tick
  count. The animation starts whenever you first call `next`
* This no longer wastes time on previous stages of animation. Ruby pauses the
  enumerator between frames and resumes it right where it left off
* We can easily construct an array of enumerators to manage a large collection
  of simultaneous animations, all running independently

But we're not done yet. Now that we can easily start independent animations,
DragonRuby's convention of basing animations on tick counts is too restrictive.
Many animations don't care how many tick counts came before they started or what
tick count it is now, they only care about how far along they should be in their
particular state change. So a new tool is needed:

```ruby
def eease duration, easing_func, &blk
  Enumerator.new do |yielder|
    if duration == 1
      yielder << blk[easing_func[1.0]]
    else
      last_i = (duration - 1).to_f
      (0...duration).each do |i|
        yielder << blk[easing_func[i / last_i]]
      end
    end
  end
end
```

This is similar to `ecount` except we always yield a value from 0 to 1,
representing the progress through the animation, passed through an easing
function. An easing function is a function that takes an input from 0 to 1 and
outputs something from 0 to 1, such as `GTK::Easing.quad`. The idea is that
`eease` is the enumerator version of DragonRuby's `ease`.

Instead of taking a start time it starts whenever you first call `next`, it
doesn't need a tick count because it tracks that internally, it still gets a
duration and an easing function, and instead of returning its progress, it yields
it so you can encapsulate the entire animation in the construction of the
enumerator. Using it looks like this:

```ruby
args.state.animation ||= eease(2.seconds, GTK::Easing.method(:quad)) { |t|
  # you'll see the progress slowly accelerate over time, due to Easing.quad
  puts "Animation is #{(t * 100).floor}% complete"
}
```

We have what we need to define the individual pieces of our animation, but now
we want to wrap it up in one parent animation, with the parent mainly delegating
to the child animations, but still doing some extra work like setting up the
initial state and looping the whole thing. This will be handy:

```ruby
class Enumerator::Yielder
  def run enum
    enum.each { |v| self << v }
  end
end
```

Now a single enumerator's yielder can go off on a side quest to run another
animation to completion before continuing on to the next stage of animation.
Combined with the built-in [Enumerator::+][plus] that concatenates enumerators
to run one after the other, finally we can rewrite our original animation:

[plus]: https://ruby-doc.org/3.2.0/Enumerator.html#method-i-2B

```ruby
args.state.animation ||= Enumerator.new { |yielder|
  args.state.sprite_pos = 320
  loop do
    yielder.run(
      ecount(0.5.seconds) +
      eease(1.seconds, easing_func) { |t|
        args.state.sprite_pos = t.lerp(320, 960)
      } +
      ecount(0.5.seconds) +
      eease(1.seconds, easing_func) { |t|
        args.state.sprite_pos = t.lerp(960, 320)
      }
    )
  end
}

args.state.animation.next
```

This now reads very similarly to our original English description without
needing any comments! I haven't even mentioned some of the bonus benefits yet.
Now that we can uniformly represent all of our animations as `Enumerator`
objects, we get fun new options like easily delaying frames (skip a call to
`next`) to simulate lag, easily skipping frames (call `next` twice) to speed up
animations after the fact, easily reset or replace animations while they are
running (reassign the variable that stores the animation), etc.

I'm still playing around with this pattern and exploring the possibilities, but I
would love to see something like this added to [DragonRuby's open source
module][oss] at some point. I may do it myself.

[oss]: https://github.com/DragonRuby/dragonruby-game-toolkit-contrib

But in the meantime, you can accomplish a lot with just `ecount`, `eease`, and
`Enumerator::Yielder::run`. All of the code in this blog post is licensed
[CC0][CC0] so go forth and make stuff!

[CC0]: https://creativecommons.org/publicdomain/zero/1.0/
