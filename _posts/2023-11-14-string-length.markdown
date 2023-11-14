---
layout: post
title: "A universal definition of string length"
categories: etc
---

There are many posts on the Internet discussing the complexity of computing
string length these days, but few making a clear recommendation of the right
thing to do. In some sense, that's probably because there is no single right
answer.

But when it comes to free-form text (text which can contain non-English
characters, emojis, etc.), I would argue that there is one very good answer and
a bunch of crappy ones. So I'll skip straight to giving you that good answer
now:

## The one good universal definition of free-form string length

When represented as valid Unicode, you should count **the number of [Unicode
scalar values][]** in the string.

[Unicode scalar values]: https://www.unicode.org/glossary/#unicode_scalar_value

You may want to handle control characters in a special way, but we'll get into
that later.

## Why do you need a universal definition of string length?

If your software ingests strings that need to be saved somewhere, you should
care about string length. Remember that a string can contain the entire
collected works of Shakespeare. Think about what happens if all of your strings
are that size.

And unless you have the luxury of writing all of your software in a single
language, running on a single platform, with a single point of data entry, you
need to worry about how the same string can be represented in different ways
depending on the circumstances. If you have two pieces of software that can each
input a string that gets saved in the same database, you don't want them
counting the length in different ways. Then you risk one piece of software being
able to load strings from the database that it won't let user the write back to
it. That would be annoying.

So why is this not trivial? Because everyone wants to measure string length in
"characters". But what is a "character" anyway?

1. In most software you can count characters in a string by moving a cursor
   through the string or trying to select portions of the string and seeing what
   can/cannot be selected. What this _actually_ counts is [grapheme clusters][]:
   the "horizontally segmentable" parts of the string.
2. Each grapheme cluster is a sequence of 1 or more [graphemes][]. Annoyingly,
   the Unicode standard calls these "characters", which I am highly suspicious
   of. Most modern software allows users to input grapheme clusters with a
   single click or tap, and does not clearly subdivide them into graphemes.
3. Each grapheme is a sequence of 1 or more [code points][], which are the
   things you can actually look up in the Unicode tables e.g., `U+1F4A9`
4. Some code points ([surrogates][] and [noncharacters][]) are used only in
   certain encodings or for internal Unicode use and never represent characters
   in Unicode. All other code points are [Unicode scalar values][].
5. Each Unicode scalar value is represented by 1-4 bytes (depending on the
   encoding)

[grapheme clusters]: https://www.unicode.org/glossary/#grapheme_cluster
[graphemes]: https://www.unicode.org/glossary/#grapheme
[code points]: https://www.unicode.org/glossary/#code_point
[surrogates]: https://www.unicode.org/glossary/#surrogate_code_point
[noncharacters]: https://www.unicode.org/glossary/#noncharacter

So there are 5 different ways to count string length: grapheme clusters,
graphemes, code points, scalar values, and bytes. Why did I pick scalar values?
Anything above code points is a non-starter: graphemes and grapheme clusters
can contain an arbitrary number of code points and thus an arbitrary number of
bytes! A length limit is pointless if a single "character" can be arbitrarily
large. 3 and 5 are also no good because they are encoding-dependent and thus
not universal: strings usually have different lengths when encoded in UTF-8 and
UTF-16, and may contain different code points!

That's why counting Unicode scalar values is the one good measure of string
length. The nice thing is that _most_ characters in _most_ languages are
represented with individual scalar values, so _most_ of the time your users
won't be too confused by this method of counting. They may just need to get used
to certain fancy characters like emojis consuming more of their string length
than others.

## Control codes

[C0][] and [C1][] are portions of the Unicode table containing control codes.
Technically these are scalar values like any other and can be counted normally,
but if you are specifically handling user-created text as opposed to a terminal
user interface you probably don't want them.

[C0]: https://www.unicode.org/charts/PDF/U0000.pdf
[C1]: https://www.unicode.org/charts/PDF/U0080.pdf

Most of them exist purely to support backwards compatibility with cryptic legacy
systems e.g., `U+009D` "operating system command". Some are useful but have
better Unicode alternatives. For example the ambiguous "new line" which often
uses some combination of the control codes `U+000A` and `U+000D` but can better
represented with `U+2028` "line separator" and `U+2029` "paragraph separator".
In my applications, I simply strip control codes from the text and calculate the
string length after.

## How to count Unicode scalar values

It is easy but often not obvious how to count scalar values. This happy family
will help us out: `👨‍👨‍👦‍👦`. This family contains 7 scalar
values (the right answer), but contains 1 grapheme cluster (on most modern
software), 11 code points in UTF-16, 22 bytes in UTF-16, and 25 bytes in UTF-8.

Here's how to do it right (and wrong) in a bunch of languages. If you know how
to do this in a language I didn't include here, please leave a comment!

### C\#

```c#
str.EnumerateRunes().Count() // 👍
str.Length // ❌ number of UTF‑16 code points
```

### Go

```go
len([]rune(str)) // 👍
len(str) // ❌ number of bytes when UTF‑8 encoded
utf8.RuneCountInString(str) // 👍 also works
```

### JavaScript

```js
[...str].length // 👍
str.length // ❌ number of UTF‑16 code points
```

### Kotlin (JVM)

```kotlin
str.codePointCount(0, str.length) // 👍
str.length // ❌ number of UTF‑16 code points
```

### Python

```python
len(str) // 👍
```

### Ruby

```ruby
str.length // 👍
```

### Swift

```swift
str.unicodeScalars.count // 👍
str.count // ❌ number of grapheme clusters
```
