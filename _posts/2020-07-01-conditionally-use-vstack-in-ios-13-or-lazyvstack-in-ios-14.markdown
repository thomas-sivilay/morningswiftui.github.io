---
layout: post
title:  "Conditionally use VStack in iOS 13 or LazyVStack in iOS 14"
date:   2020-07-01 17:38:05 +1000
categories: swiftui
tags: vstack lazyvstack ios14
---
**WWDC2020** offered a new suite of Views, one interesting one is `LazyVStack` which allow us to delay the initialization of some content only when needed. Unfortunately it's only available in iOS 14. **What if you still want to offer an app that support iOS 13?**

You can easily achieve this by creating your own wrapper. Here's how we can offer either a `VStack` in iOS 13 and the new shiny `LazyVStack` in iOS 14:

```swift
public struct MyVStack<Content>: View where Content : View {

  let content: () -> Content
  let alignment: HorizontalAlignment
  let spacing: CGFloat?

  public init(alignment: HorizontalAlignment = .center, spacing: CGFloat? = nil, @ViewBuilder content: @escaping () -> Content) {
    self.content = content
    self.alignment = alignment
    self.spacing = spacing
  }

  @ViewBuilder public var body: some View {
      if #available(iOS 14.0, *) {
          LazyVStack(alignment: alignment, spacing: spacing, content: self.content)
      } else {
          VStack(alignment: alignment, spacing: spacing, content: self.content)
      }
  }
}
```

With this quick trick, we now can use `MyVStack` following the same API as Apple provides us. We still have the ability to adjust the horizontal adjustment, the spacing and to pass the content in a similar shape.

Of course, this isn't limited to `VStack` and `LazyVStack`. We could extend this to `HStack` and his `LazyHStack` counter part.
