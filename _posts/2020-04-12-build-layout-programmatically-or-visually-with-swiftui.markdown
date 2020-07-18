---
layout: post
title:  "Build layout programmatically or visually with SwiftUI"
date:   2020-04-12 17:38:05 +1000
categories: swiftui
tags: declarative syntax, canvas
redirect_from: "/build-layout-programmatically-or-visually-with-swiftui"
---
**How do you build your layout?** Are you using auto layout? Frame calculations? Interface builder? I think we’ve all try different approaches and they all come with tradeoffs. It’s even harder when working within a group of people because sometimes the choice has been made out of common beliefs that became a convention. I believe it’s difficult to satisfy everyone because it’s just a matter of taste and preference.. until SwiftUI?

![Poll]({{ site.baseurl }}/assets/images/20200412-poll.png)

Out of the numerous ways of building a layout I listed, I think we could group them into two different categories. Either programmatically or visual? Which might be the same as, does your brain prefer to read/write or to picture something?

## Programmatically

```swift
HStack(spacing: 0) {
  ZStack {
    Circle()
      .frame(width: 32, height: 32)
      .foregroundColor(type.backgroundColor)
    Image(systemName: type.iconName)
      .frame(width: 20, height: 20)
      .foregroundColor(type.iconColor)
  }
  Text(type.title)
    .font(.headline)
    .foregroundColor(.secondary)
    .multilineTextAlignment(.center)
    .lineLimit(1)
    .padding(.horizontal, 12)
}
```

There is less efforts required into translating the SwiftUI syntax into a visual representation of it. It is easier to grasp the intention of a block of code. So it’s definitely programmatically! **We write and read from there.** After all, that’s how Apple described SwiftUI.

## Visually

Even with an improved syntax, *Apple* didn’t stop there and instead, decided to give us `PreviewProvider`. With few more lines of codes, that also allow us to inject different variables we can have a live visual representation of what we can read and write.

But that’s not it… I almost forgot about it because of habits of doing layout programmatically:

- in the **Canvas** (*the part of Xcode where we have a live representation*) you can select existing Views and with the inspector, like on interface builder, you can adjust values and add more modifiers to them
- additionally, if you want to add new element to your View, you can drag and drop elements from the **Library** button (+) in the toolbar.

![Visual]({{ site.baseurl }}/assets/images/20200412-visual.png)

## The best of both worlds

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">How do you build user interface with SwiftUI? <a href="https://twitter.com/hashtag/SwiftUI?src=hash&amp;ref_src=twsrc%5Etfw">#SwiftUI</a><a href="https://twitter.com/hashtag/swift?src=hash&amp;ref_src=twsrc%5Etfw">#swift</a></p>&mdash; Thomas Sivilay (@thomassivilay) <a href="https://twitter.com/thomassivilay/status/1249123120683929602?ref_src=twsrc%5Etfw">April 11, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


**Providing both options gives flexibility and reduce trade-offs**, it allow us to choose the best tool for based on our skills, preferences and a situation. If you prefer the visual way, it doesn’t force the others that prefers the programmatically way to read something else than code (like a XIB). The source of truth is still code because while updating visually, it generates the required code.

Even more than the best of both worlds, I believe with SwiftUI’s principles, the productivity is increased with the ability to quickly jump to a specific view, quickly iterate on it, see live update with injected data/state/environment. The feedback loop is shorter, we don’t need to navigate to a particular view to verify that our tiny change is accurate, we have some kind of hot reload.
