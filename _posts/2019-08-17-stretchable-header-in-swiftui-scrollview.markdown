---
layout: post
title: "Stretchable header in SwiftUI ScrollView"
date:   2019-08-17 17:38:05 +1000
categories: swiftui
tags: scrollview header preferencekey experiment
redirect_from: "/stretchable-header-in-swiftui-scrollview"
---
SwiftUI doesn’t provide yet a `TableHeaderView` or something similar to `UIScrollViewDelegate` but I tried to experiment with different Views to have something close to a stretchable header.

![header]({{ site.baseurl }}/assets/images/20190817-header.png)

The initial idea is to try to implement a stretchable header that would play well with a `List` or a `Form`.

And if I want to describe what I would like to achieve it will have been as simple as this:

```swift
struct Example: View {

    let minHeight: CGFloat = 200
    @State var stretchableHeight: CGFloat = minHeight

    var body: some View {
        VStack {
            StretchableHeader().frame(height: stretchableHeight)
            List()
                .onScroll { offset in
                    self.strechableHeader = max(minHeight, minHeight + offset.y)
                }
        }
    }
}
```

## Not that simple

Unfortunately this doesn’t work, we don’t have an extension of View with a `ViewModifier onScroll` which will help us achieve our goal. Even if we did have this, I’m not sure if it will play nicely because we are mutating the height of the `StretchableHeader` from List.

## The key is to use PreferenceKey

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Finally got something! Wanted to play with &quot;stretchy&quot; header in a scroll view with <a href="https://twitter.com/hashtag/SwiftUI?src=hash&amp;ref_src=twsrc%5Etfw">#SwiftUI</a>. The key is.. preferenceKey! Still needs a lot of tweaks but it&#39;s good to see some progress. <a href="https://twitter.com/hashtag/ScrollView?src=hash&amp;ref_src=twsrc%5Etfw">#ScrollView</a> <a href="https://twitter.com/hashtag/ContentOffset?src=hash&amp;ref_src=twsrc%5Etfw">#ContentOffset</a> <a href="https://twitter.com/hashtag/PreferenceKey?src=hash&amp;ref_src=twsrc%5Etfw">#PreferenceKey</a> <a href="https://t.co/cunjfBLD65">pic.twitter.com/cunjfBLD65</a></p>&mdash; Thomas Sivilay (@thomassivilay) <a href="https://twitter.com/thomassivilay/status/1161397525355458560?ref_src=twsrc%5Etfw">August 13, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

After some research and not finding much help, there is actually one blog that helped a lot in this experimentation: https://swiftui-lab.com

And they will probably soon share how to achieve similar goal, hopefully in a more complete way:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Here&#39;s another example of the <a href="https://twitter.com/hashtag/SwiftUI?src=hash&amp;ref_src=twsrc%5Etfw">#SwiftUI</a> PreferenceKey protocol in action. Maybe one day ScrollView will support refresh controls natively... until then, here&#39;s my first approximation. Blog post + source code coming soon. First I need to do some cleanup. <a href="https://t.co/CbcymKTyIG">pic.twitter.com/CbcymKTyIG</a></p>&mdash; The SwiftUI Lab (@SwiftUILab) <a href="https://twitter.com/SwiftUILab/status/1162355929230315520?ref_src=twsrc%5Etfw">August 16, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Anyway, `PreferenceKey` was what helped me having something closer to the initial goal, you can read about it in [https://swiftui-lab.com/communicating-with-the-view-tree-part-1/][0]

I use `PreferenceKey` as a way to share some values between `Views` **that share the same ancestors**. So in this example the value that I want to share is “what is the offset on the scroll” from the List to the StretchableHeader.

## Some solution

Instead of embedding everything in a VStack, the first change is to use a `ZStack` so we can have the `StretchableHeader` to be displayed on top of the `ScrollViewContentView`. We also need to adjust the offset of the content based on the height of header.

We use `@State` as our local variable to capture both the current offset of the scroll and the default height of the header.

```swift
struct Example: View {

        @State var yOffset: CGFloat = 0
        let headerHeight: CGFloat = 100

        var body: some View {
            ZStack {
                StretchableHeader(offset: $yOffset, headerHeight: headerHeight)
                    .zIndex(2)
                ScrollViewContentView(offset: $yOffset)
                    .offset(y: headerHeight)
            }
            .onPreferenceChange(ScrollOffsetPreferenceKey.self) {
                self.yOffset = $0.first ?? 0
            }
        }
    }
```

Breaking this example into two View brings more clarity but involve injecting the state as a @Binding.

```swift
struct StretchableHeader: View {

        @Binding var yOffset: CGFloat
        let headerHeight: CGFloat

        var body: some View {
            GeometryReader { proxy -> AnyView in
                return AnyView(
                    ZStack {
                        Image(“catalina”)
                            .frame(height: max(self.yOffset, self. headerHeight))
                            .clipped()
                        Text(“Content offset: \(self.offset)”)
                    }
                    .frame(width: proxy.size.width, height: max(self.yOffset, self.headerHeight))
                    .clipped()
                )
            }
        }
    }
```

We use `GeometryReader` to have access to a `GeometryProxy` to help us defining the exact frame of the StretchableHeader where we keep its width but set the height to use the yOffset so that the content will grow.

Earlier, we said we will use a PreferenceKey to define and share the data representing the scroll offset.

```swift
struct ScrollOffsetPreferenceKey: PreferenceKey, Equatable {
        static var defaultValue: [CGFloat] = []

        static func reduce(value: inout [CGFloat], nextValue: () -> [CGFloat]) {
            value.append(contentsOf: nextValue())
        }
    }
```

Which is already listened/handled in the ExampleView with:

```swift
@inlinable public func onPreferenceChange<K>(_ key: K.Type = K.self, perform action: @escaping (K.Value) -> Void) -> some View where K : PreferenceKey, K.Value : Equatable
```

But needs to be mutated in ScrollViewContentView using:

```swift
@inlinable public func preference<K>(key _: K.Type = K.self, value: K.Value) -> some View where K : PreferenceKey
```

The way I choose to update the PreferenceKey values if by finding the minY value of the first view inside ScrollViewContentView.

```swift
struct ScrollViewContentView: View {

        let correction: CGFloat = 85 + 44
        @Binding var offset: CGFloat

        var body: some View {
            GeometryReader { proxy -> AnyView in
                return AnyView(
                    Form {
                        GeometryReader { firstViewProxy in
                                Text(“First view”)
                                    .preference(key: ScrollOffsetPreferenceKey.self,
                                                value: [firstViewProxy.frame(in: .global).minY - self.correction])
                        }
                        Text(“Second view”)
                    }
                    .frame(width: proxy.size.width)
                )
            }
        }
    }
```

Again GeometryReader help us finding the minY which we use as a value update of ScrollOffsetPreferenceKey.

## Known issues

### Correction for navigation bar height

Maybe you noticed that in the ScrollViewContentView there is a correction value, which I’m not comfortable with and needs to change based on the displayMode of a .navigationBarTitle.

### Large title isn't my friend

With a large title, we will need to modify the offset of the StretchableHeader based on the scrollOffset but by doing so we will again update the scrollOffset.. Then we have a nice infinite loop.


### Using first view isn't a good idea

Using the first view to update the preference value might work for our example here, it will also play nicely if we want to have a refresh control in the stretchable header. But if we have a bigger header height, you can find some issues where we are not updating the preference key values anymore. I believe it’s because of the first view not being on the screen which will not relay frame changes anymore!

## So what's next?

That was just some experimentation that I wanted to share, definitely not something I recommend but some ideas to that could be use to improve and solve the initial goal. Without much documentation and using SwiftUI at the early stage, we do with what we have and the most important is that I had fun and learned more about GeometryReader and PreferenceKey trying this.
