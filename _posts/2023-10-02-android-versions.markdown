---
layout: post
title: "Android Versions"
categories: etc
---

So you want to write Android apps? Well first you have to understand the
incredibly complicated world of versions in Android. Start by answering these
questions.

1. What version of the [Kotlin language][kotlin] do you want to write in? You
   should definitely be writing in Kotlin. We'll call this your **Kotlin
   version**.
2. What version of the [Java language][java] do you want to write in? You really
   shouldn't be writing Java but sometimes you gotta. Just the major version is
   enough. We'll call this your **Java version**.
3. What's the oldest [Android][android] version you want your app to run on?
   Great. I don't care. Instead find the oldest _Android API level_ you want
   your app to run on (e.g. Android 13 is API level 33). We'll call this your
   **minimum Android version**.
4. What's the newest Android version you're actually going to test your app on?
   Again, I don't care. Get that version's Android API level. That's your
   **target Android version**.

[kotlin]: https://kotlinlang.org/docs/releases.html
[java]: https://www.java.com/releases/
[android]: https://developer.android.com/tools/releases/platforms

## Fill in the basics

### Kotlin

Use your **Kotlin version** as the version of the kotlin gradle plugin applied
in your build script:

```kotlin
plugins {
    id("org.jetbrains.kotlin.<...>") version "whatever"
}
```

`<...>` is probably gonna be `android` or `multiplatform` depending on how cool
you are.

A general word of warning: this is the _only_ place the Kotlin version must
appear. Other Kotlin plugins/libraries may look like they share this version but
that is only a loose convention. If those plugins/libraries have bugs and need
to be patched they will intentionally diverge from the language version! Don't
reuse the Kotlin version in your build script for other things.

### Java

Use your **Java version** to fill in these parts of your gradle build script:

```kotlin
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_WHATEVER
        targetCompatibility = JavaVersion.VERSION_WHATEVER
    }
    kotlinOptions {
        jvmTarget = "WHATEVER"
    }
}
```

I love how these are totally different types in the script.

These versions should always match! If you need to use a different Java version
for some of your code you need to split it into a separate gradle project so it
gets its own build script and can get its own versions.

Note that this only affects how you write and build your Java code. Because of
Android's [desugaring][desugar] process, your app can still run on Android
versions with older JVMs. The compiler will let you know if you pick a version
that's too new.

[desugar]: https://developer.android.com/studio/write/java8-support

### Android

Your **minimum and target Android versions** also go in the build script:

```kotlin
android {
    defaultConfig {
        minSdk = 69
        targetSdk = 69 // can be greater than minSdk
    }
}
```

## The rest

Now you need to pick a bunch of other versions, all constrained ultimately by
the answers you chose above.

### Android Compile SDK

When you compile your Android app, it will compile it against a certain version
of the API. This should be **greater than or equal to your target Android
version**. Technically you can always choose the latest API version, but
practically it never needs to be greater than your target version. Choosing a
larger version gives you access to the newer APIs earlier, letting you decouple
your development cycle slightly from Android's release cycle.

Whatever version you choose, you must specify it in your gradle build script.
While you're at it, you also get an easy one: you need build tools and those can
pretty much always be the latest version since they're usually backwards
compatible, so just look up the latest [build tools][buildtools] and specify it.
You can use older tools, but not older than the compile SDK version.

[buildtools]: https://developer.android.com/tools/releases/build-tools#notes

```kotlin
android {
    compileSdk = 69 // can be greater than targetSdk
    buildToolsVersion = "420"
}
```

Technically you can leave out build tools and one will be picked for you but
it's a better idea to be explicit so you can easily pull in a patch if one is
released.

### Android Gradle Plugin (AGP)

If you've watched [my talk about gradle][gradle] you know that it isn't so much
a build tool as it is a tool for making build tools. AGP is the _actual_ build
tool for building Android apps. Your script needs to apply either the
`com.android.application` or `com.android.library` plugin. You want the latest
version of this plugin since newer versions can fix build issues and improve
build performance. Hopefully there isn't a reason you can't use the newest
version. Foreshadowing.

[gradle]: https://www.youtube.com/watch?v=e1BQeYlKOgA

```kotlin
plugins {
    id("com.android.application") version "whatever"
}
```

### Gradle

Oh yeah, you needed gradle set up. Why are we only getting to it now? Because
you don't care. You want the latest version of gradle so that your build runs
super fast and doesn't have any compiler issues. Hopefully there isn't a reason
you can't use the newest version. Foreshadowing.

You should be using gradle wrapper and setting the version in
`gradle-wrapper.properties`:

    distributionUrl=https\://services.gradle.org/distributions/gradle-whatever-bin.zip

### Android Studio

Yeah you need Android Studio too. Again, latest version.

## Dependency Hell

And now all the foreshadowing pays off. Everything is broken! 💥

Well, maybe. Why? Because AGP, Gradle, Android Studio, Kotlin, and Java can all
be incompatible with each other. Yay! There's no simple answer here. You have to
go back and start downgrading stuff until it starts working or, if you can't
find a valid combination, change one of your original four answers to make it
possible. Good luck.

The most important things to keep consistent here are Kotlin, Java, AGP, and
your Android versions. If you get them wrong, you won't necessarily see a
spectacular failure at build time; your app may fail spectacularly only at
runtime! Or it may only fail spectacularly when you build a release optimized
for the Play Store! Be very careful when changing the versions of _any_ of these
things.

The [AGP release notes][agp] are pretty good at capturing possible
incompabilities.

[agp]: https://developer.android.com/build/releases/gradle-plugin

## More dependency hell

The story continues with every library you include. Want to grab the latest
version of some library? Well that might require you increasing your Java target
compatibility, which means you need to update Kotlin, which means you need to
update AGP, which means you need to update Gradle, which means... etc.
