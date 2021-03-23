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

* `Optional`. Means "it is present or not". This doesn't work because the data
  are _always_ present in our use case.
* `Result`. Means "it is preset or is an error". This doesn't work because our
  error is always the same--that the deserialized value was not understood--and
  we need to specifically maintain the "bad" value rather than allowing any
  error type. Also `Result` is not `Codable`, meaning we can't avoid boilerplate
  if we want to use it in our `Codable` types.

What we want is an enum with two cases:

1. Recognized, meaning we see that the value is one that our program is designed
   to handle
2. Unrecognized, meaning it is some other value

It turns out that writing this generic type in Swift is super easy and useful,
so here you go.

## Implementation

All of these methods follow the same trivial pattern: use the known strict type
if possible, otherwise use the unrecognized raw value.

```swift
public enum Recognizable<Known: RawRepresentable>: RawRepresentable {
    case recognized(Known)
    case unrecognized(Known.RawValue)

    public var rawValue: Known.RawValue {
        switch self {
        case .recognized(let recognized):
            return recognized.rawValue
        case .unrecognized(let value):
            return value
        }
    }

    // N.B. this constructor is non-failable unlike most RawRepresentable types
    public init(rawValue: Known.RawValue) {
        guard let recognized = Known(rawValue: rawValue) else {
            self = .unrecognized(rawValue)
        }
        self = .recognized(recognized)
    }

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
        let rawValue: Known.RawValue
        switch self {
        case .recognized(let recognized):
            rawValue = recognized.rawValue
        case .unrecognized(let value):
            rawValue = value
        }
        var container = encoder.singleValueContainer()
        try container.encode(rawValue)
    }
}
```

## Example

Take this simple example of loading some JSON

```swift
enum Animal: String, Codable {
    case cat, dog, hamster
}

struct Pet: Codable {
    let name: String
    let type: Animal
}

let json = """
{"name":"Liz","type":"lizard"}
"""

let pet = try decoder.decode(Pet.self, from: json.data(using: .utf8)!)
```

Currently this is very strict, it will throw because we do not have a `lizard`
case for `Animal`. If we want to write some catch-all code that handles all unrecognized
types of pets, the only simple way to do this is normally to declare `type:
Animal?` (and throw away the unrecognized value), or add a bunch more code to do
custom decoding.

But now we can simply declare `type: Recognizable<Animal>` and the rest is free.
