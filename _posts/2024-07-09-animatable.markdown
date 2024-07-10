---
layout: post
title: "Animatable Content in SwiftUI"
categories: etc
---

Animations in SwiftUI are pretty magical.
For example, say you have a progress bar that you expect to have a big jump in progress:

```swift
struct SwiftUIView: View {
    @State var progress = 0.0
    var body: some View {
        Button("0%") { progress = 0.0 }
        Button("100%") { progress = 1.0 }
        ProgressView(value: progress)
    }
}
```

When the value changes, the progress immediately jumps all the way to 100 or back to 0.
Not very sexy.
Magically, this simple change fixes it:

```swift
ProgressView(value: progress)
    .animation(.default, value: progress)
```

This tells SwiftUI to animate any changes to the progress bar whenever the `progress` value changes.
But what about that `.default` parameter?
That is the type of animation to use; examples of other built-in values are `.bouncy` and `.easeInOut`.
Even sexier!

But if you try them out, you'll see they make no difference compared to `.default`.
What gives?

The problem is that it is up to each view to decide how to animate itself.
This is where the magic ends: `ProgressView` doesn't pay attention to the type of animation at all.
It just checks whether it's been told to animate or not.

But in this case, we have a potential workaround: we just need the progress value itself to animate.
If the progress smoothly transitioned between 0 and 1, we could pass the transitional values to the `ProgressView` and it would look smooth.
In other words, we don't really need to animate the whole `ProgressView`, just its content.
This is what the `Animatable` protocol solves.

To fix this, we will wrap the `ProgressView` in an `Animatable` view that explicitly says what property to animate:

```swift
struct AnimatableProgressView: View, Animatable {
    var value: Double
    var animatableData: Double {
        get { value }
        set { value = newValue }
    }

    var body: some View {
        // prevent negative values (they show a spinner instead)
        ProgressView(value: value < 0.0 ? 0.0 : value)
    }
}
```

This is still a little magical.
`animatableData` is a protocol override that tells any animations to animate the `value` property.
Now, the `ProgressView` itself doesn't need any animations; it just blindly uses `value` which is itself animating.

Note that it is actually important to enforce the lower bound.
Animation curves are allowed to overshoot their target, for example `.bouncy` could briefly animate negative numbers into the `value` variable when transitioning to 0.
Since the default behavior of `ProgressView` is to show a spinner for a negative progress, that would look bad.

## General use

This pattern can be generalized to any view that doesn't animate as you expect but which you can manually update just by setting new values.
Create a new view which is `Animatable` and which has `var`s for whatever data you want to animate.
Declare `animatableData` to get/set the data.

Note that the type of `animatableData` must conform to the [VectorArithmetic](https://developer.apple.com/documentation/swiftui/vectorarithmetic) protocol.
This can be accomplished with an extension if the type you want doesn't already conform.
If you have two separate values you want to animate on the same view, you can use `AnimatablePair` to group them together.
If you have more than two values to animate, I recommend putting them all in one struct and making the whole struct conform to `VectorArithmetic` in the naïve way: component-by-component.
