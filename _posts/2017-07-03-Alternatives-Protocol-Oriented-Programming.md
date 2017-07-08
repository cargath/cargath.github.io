---
layout: post
title: "Alternatives to Protocol-Oriented Programming"
date: 2017-07-03
---

We all know that [composition is better than inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance). Apple made a big push in this direction at WWDC15 by officially declaring protocols the next big paradigm in programming. If you haven't already seen it, go watch [Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/wwdc/2015/?id=408), it's definitely worth your time.

Like someone with a new hammer i immediately went trying to hit a particular nail, a problem that everyone trying to make a game for the first time has to deal with sooner or later: How to compose your game objects without duplicating code while avoiding deep inheritance hierarchies?

Consider the following example:

```swift
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

let birds: [Bird] = [Penguin(), Eagle()]

for bird in birds {
    if let bird = bird as? Flyable {
        bird.fly()
    }
}
```

This game has several birds a characters, some of which fly, some of which don't. We would like every flying bird to update its flying state. The problem: We only know the base class of our game objects, so we do a lot of typecasting, which feels like a workaround.

Turns out the solution was another way of implementing composition: An Entity-Component architecture.

```swift
protocol Component {
    func update()
}

struct Flying: Component {
    func update() {
        print("huiii")
    }
}

protocol Bird {
    var components: [Component] { get }
}

extension Bird {
    func updateAll() {
        for component in components {
            component.update()
        }
    }
}

struct Penguin: Bird {
    let components: [Component] = []
}

struct Eagle: Bird {
    let components: [Component] = [Flying()]
}

let birds: [Bird] = [Penguin(), Eagle()]

for bird in birds {
    bird.updateAll()
}
```

Apple actually provides [helper classes for this paradigm in GameplayKit](https://developer.apple.com/library/content/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1), also released at WWDC15. Although that [session](https://developer.apple.com/videos/play/wwdc2015/608/) got a lot less attention. At my day-job we used Entity-Component to [break up an AppDelegate](http://geme.github.io/swift/2016/04/28/Refactoring-AppDelegate.html), so this doesn't only apply to games 😉

I guess my point here is, that there is no [silver bullet](http://chris.eidhof.nl/post/protocol-oriented-programming/). You should always choose the tool that's best suited for your problem, not try to bend your tool to solve your problem.
