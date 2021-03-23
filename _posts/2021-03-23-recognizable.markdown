---
layout: post
title: "Recognizable types in Swift"
date: 2021-03-23
categories: etc
---

I love `Codable` in Swift, especially with enums. It lets you define
a limited set of values for deserialization without a ton of boilerplate.
However I've had one problem pop up a few different times with it: what if you
want to decode strictly, but also provide a data-preserving catch-all case?

Usually this occurs when you are deserializing something but also passing the
data on to some other framework/API/etc. You want your code to handle it
strictly in order to avoid corner cases, but throwing away the bad data is not
an option. Swift has existing wrapper enum types which are close to want we
want, but not quite right:

* `Optional` means "it is present or not". This doesn't work because the data
  are _always_ present in our use case.
* `Result` means "it is preset or is an error". This doesn't work because our
  error is always the same--that the deserialized value was not understood--and
  we need to specifically maintain the "bad" value rather than allowing any
  error type. Also `Result` is not `Codable`, meaning we can't avoid boilerplate
  if we want to use it in our `Codable` types.

What we want is an enum with two cases:

1. Recognized, meaning we see that the value is one that our program is designed
   to handle
2. Unrecognized, meaning it is some other value

## Example

I want to be able to write code like this

```swift
enum Pet: String, Codable {
    case cat, dog
}

// always non-optional, possibly using a catch-all
let rawPet = Recognizable<Pet>(rawValue: "fish")
// only throws if data can't be decoded as a String
let jsonPet = try decoder.decode(Recognizable<Pet>.self, from: data)
```

And then I can define separate code paths for the catch-all case

```swift
guard case let .recognized(known) = pet else {
    NSLog("Can't handle unknown pet \(pet.rawValue)")
    return
}
// do something with known
```

Or if some code doesn't care about the catch-all case you can treat it as
optional when needed:

```swift
let typeName = pet.recognized?.rawValue ?? "some unknown pet type"
```


It turns out that writing this generic type in Swift is super easy and useful,
so here you go.

## Implementation

All of these methods follow the same trivial pattern: use the known strict type
if possible, otherwise use the unrecognized raw value.

```swift
public enum Recognizable<Known: RawRepresentable>: RawRepresentable {
    case recognized(Known)
    case unrecognized(Known.RawValue)

    // convert to optional when you don't care about unrecognized
    public var recognized: Known? {
        switch self {
        case .recognized(let known):
            return known
        case .unrecognized(_):
            return nil
        }
    }

    public var rawValue: Known.RawValue {
        switch self {
        case .recognized(let known):
            return known.rawValue
        case .unrecognized(let value):
            return value
        }
    }

    // N.B. this constructor is non-failable unlike most RawRepresentable types
    public init(rawValue: Known.RawValue) {
        guard let recognized = Known(rawValue: rawValue) else {
            self = .unrecognized(rawValue)
            return
        }

        self = .recognized(recognized)
    }

    // less verbose construction if you have the Known type already
    public init(_ recognized: Known) {
        self = .recognized(recognized)
    }
}

extension Recognizable: Decodable where Known: Decodable, Known.RawValue: Decodable {
    public init(from decoder: Decoder) throws {
        do {
            self.init(try Known(from: decoder))
        } catch {
            let container = try decoder.singleValueContainer()
            self = .unrecognized(try container.decode(Known.RawValue.self))
        }
    }
}

extension Recognizable: Encodable where Known: Encodable, Known.RawValue: Encodable {
    public func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        switch self {
        case .recognized(let known):
            try container.encode(known)
        case .unrecognized(let value):
            try container.encode(value)
        }
    }
}
```

Enjoy!
