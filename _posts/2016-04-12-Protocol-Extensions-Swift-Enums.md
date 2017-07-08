---
layout: post
title: "Protocol extensions for Swift enums"
date: 2016-04-12
---

Today i got into a situation where i realised that i was writing the same extensions for various enums over and over again.

Let's say we're parsing JSON. JSON often promises to only return specific values for a key, like an enum, but we're actually parsing strings. So we would define an enum like this:

```swift
public enum Foobar: String {
    case foo = "FOO"
    case bar = "BAR"
}
```

And then parse the JSON string like this:

```swift
let blub = Foobar(rawValue: "FOO")
```

Sometimes we would like the parser to act a little more fuzzy, so we do:

```swift
extension Foobar {
    public static func fromStringIgnoringCase(string: String) -> Foobar? {
        return Foobar(rawValue: string.uppercaseString)
    }
}
```

Let's do this more elegantly and make it reusable. Instead of extending all enums with a `String` raw value (which, to my knowledge, is currently not possible in Swift), we can make our enums with `String` raw values conform to a type like:

```swift
public protocol StringEnumType {
    var rawValue: String { get }
    init?(rawValue: String)
}

public enum Foobar: String, StringEnumType {
    case foo = "FOO"
    case bar = "BAR"
}
```

Note that the enum already conforms to the protocol, no need to write repetitive boilerplate code. Now we can just extend the `StringEnumType` protocol:

```swift
public extension StringEnumType {
    public static func fromStringIgnoringCase(string: String) -> Self? {
        return Self.init(rawValue: string.uppercaseString)
    }
}
```
