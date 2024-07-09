---
layout: post
title: "Animatable Content in SwiftUI"
categories: etc
---

struct AnimatableProgressView: Animatable, View {
    var value = 0.0
    var animatableData: Double {
        get { value }
        set { value = newValue }
    }

    var body: some View {
        // prevent negative values (they show a spinner instead)
        ProgressView(value: value < 0.0 ? 0.0 : value)
    }
}


struct ProgressAnimTest: View {
    @State var progress = 0.0

    var body: some View {
        Button("0%") {  progress = 0.0  }
        Button("100%") {  progress = 1.0  }
        Text("\(progress)")
            .animation(.default, value: progress)
        AnimatableProgressView(value: progress)
            .animation(.default, value: progress)
