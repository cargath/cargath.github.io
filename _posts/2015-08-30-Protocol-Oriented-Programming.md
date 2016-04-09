---
layout: post
title: "Protocol-Oriented Programming and the multiple inheritance problem"
date: 2015-08-30
---

I really wanted to adopt the [protocol-oriented programming](https://developer.apple.com/videos/wwdc/2015/?id=408) style for my current projects. But when i started implementing things, i quickly noticed that it doesn't actually solve any problem related to inheritance. Consider the following three examples:

    protocol Bird { }

    protocol Flyable {
        func fly()
    }

    struct Penguin: Bird { }

    struct Eagle: Bird, Flyable {
        func fly() {
            print("huiii")
        }
    }

    let birds: [Bird] = [ Penguin(), Eagle() ]

    for bird in birds {
        if let bird = bird as? Flyable {
            bird.fly()
        }
    }

This version is short and safe, but if we have lots of objects typecasting everything might be slow.

    protocol Bird {
        var flyable: Bool { get }
    }

    extension Bird {
        var flyable: Bool { return false }
    }

    protocol Flyable {
        func fly()
    }

    extension Bird where Self: Flyable {
        var flyable: Bool { return true }
    }

    struct Penguin: Bird { }

    struct Eagle: Bird, Flyable {
        func fly() { print("huiii") }
    }

    let birds: [Bird] = [
        Penguin(),
        Eagle()
    ]

    for bird in birds {
        if bird.flyable {
            (bird as! Flyable).fly()
        }
    }

This is faster, but we lose type safety.

    protocol Bird {
        var flyable: Bool { get }
        func fly()
    }

    extension Bird {
        var flyable: Bool { return false }
        func fly() {  }
    }

    protocol Flyable { }

    extension Bird where Self: Flyable {
        var flyable: Bool { return true }
        func fly() { print("huiii") }
    }

    struct Penguin: Bird { }

    struct Eagle: Bird, Flyable { }

    let birds: [Bird] = [
        Penguin(),
        Eagle()
    ]

    for bird in birds {
        if bird.flyable {
            bird.fly()
        }
    }

This is fast and safe, but now we are back to one giant base object.

I'm still unsure which version i prefer. Maybe i missed some magical solution?
