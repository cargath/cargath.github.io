---
layout: post
title: "Continuous corners in SwiftUI"
date: 2019-06-23
---

Making rounded corners and borders with rounded corners in SwiftUI is pretty simple:

```swift
struct RootView: View {

    var body: some View {
        Text("Lorem ipsum.")
            .padding()
            .background(Color.orange)
            .cornerRadius(8)
            .border(Color.black, width: 2, cornerRadius: 8)
    }

}
```

But, if you are anything like me, you want the new "continuous" corner style introduced in UIKit, the "squircle" or "superellipse" one that Apple has been using for some time now. Fortunately this is easy, too.

The `cornerRadius()` modifier is just a special case of the `clipShape()` modifier with a `RoundedRectangle`. Similarly, the `border()` modifier can be recreated by using an `overlay()` with a `stroke()`. Once you've done that, you can change the style of the `RoundedRectangle` to `.continuous`:

```swift
struct RootView: View {

    var body: some View {
        Text("Lorem ipsum.")
            .padding()
            .background(Color.orange)
            .clipShape(RoundedRectangle(cornerRadius: 8, style: .continuous))
            .overlay(RoundedRectangle(cornerRadius: 8, style: .continuous)
                .stroke(Color.black, lineWidth: 2)
            )
    }

}
```

And that's it 🙌
