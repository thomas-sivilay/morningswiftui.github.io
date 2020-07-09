---
layout: post
title:  "Show a banner from anywhere in SwiftUI using ViewModifier"
date:   2019-11-05 17:38:05 +1000
categories: swiftui
tags: viewmodifier banner
---
Creating view isn’t hard with SwiftUI, we can quickly iterate to build our final `struct BannerView: View`. But what if we would like to display it on top of our content? **What if we have multiple `View` that needs to show a banner?** Aren’t we going to duplicate code in lot of places with also defining how we want to animate in/out the transition?

## The solution

### Create a `BannerView`

It could be anything here, up to you to decide how it looks like, what do we need to show in a banner. Does it needs a Title? A subtitle? A button? For our example we will just need a data structure `BannerData` to represent what the banner needs to display.

```swift
struct BannerData {
    let title: String
    var actionTitle: String? = nil
    // Level to drive tint colors and importance of the banner.
    var level: Level = .info

    enum Level {
        case .warning
        case .success

        var tintColor: Color {
            switch self {
            case .warning: return .yellow
            case .success: return .green
            }
        }
    }
}

struct BannerView: View {
    let data: BannerData
    var action: (() -> Void)?

    var body: some View {
        HStack {
            Text(data.title)
            Spacer()
            Button(action: 
                action?()
            ) {
                Text(data.actionTitle ?? “Action”)
                    .foregroundColor(.white)
                    .padding(EdgeInsets(top: 6, leading: 12, bottom: 6, trailing: 12))
                    .background(Rectangle().foregroundColor(Color.black.opacity(0.3)))
            }
        }
        .padding(EdgeInsets(top: 12, leading: 8, bottom: 12, trailing: 8))
        .background(data.level.tintColor)
    }
}
```
In your case, you might want to use an image, a subtitle, an option to be dismissible or not. But for this example we just decided to create a simple `BannerView` .

### Display the banner the naive way

Having both `BannerData` and `BannerView` we could already display it from any other SwiftUI View.

```swift
@State private var showBanner: Bool = false
@State private var countTap: Int = 0

var body: some View {
    VStack {
        if showBanner {
            BannerView(data: BannerData(title: “Naive way”, actionTitle: “OK”, level: .warning), action: {
                self.countTap += 1
            })
            .animation(.easeInOut)
            .transition(AnyTransition.move(edge: .top).combined(with: .opacity))
            .onTapGesture {
                self.showBanner = false
            }
        }
        Text(“Some text”)
    }
}
```

I call this the *naive way* because we are doing too much:
- We know that a banner will be displayed, so we wrap everything in a `VStack`.
- We use have a local `@State var showBanner` to display it or not. 
- We define the animation and transition.

Nothing is *wrong*, it could just be improved. The syntax could be more concise and less repetitive accross your app.

### Introducing ViewModifier

From *Apple* documentation, this is what we can read about `ViewModifier`.

> A modifier that can be applied to a view or other view modifier, **producing a different version of the original value**.

The cool part is the second part of the sentence. We can produce a different version of the original value? Isn’t this what we want with a banner?

Given a `@State showBanner` we need to add a `VStack { }` to display the `BannerView` on top of the rest of the content.

So I gave it a try and this is how my ViewModifier looks like for this banner example:

```swift
struct BannerViewModifier: ViewModifier {
    @Binding var isPresented: Bool
    let data: BannerData
    let action: (() -> Void)?

    func body(content: Content) -> some View {
        VStack(spacing: 0) {
            if isPresented {
                BannerView(data: data)
                    .animation(.easeInOut)
                    .transition(AnyTransition.move(edge: .top).combined(with: .opacity))
                    .onTapGesture {
                        self.isPresented = false
                    }
            }
        }
    }
}
```

We basically moved almost everything we had before from the naive way, to this BannerViewModifier. 

And to make this accessible from any view, we just need to create an extension of View.

```swift
extension View {
    func banner(isPresented: Binding<Bool>, data: BannerData, action: (() -> Void)? = nil) -> some View {
        self.modifier(BannerViewModifier(isPresented: isPresented, data: data, action: action))
    }
}
```

This gives us the ability to **show a banner from any View**.. Any! So we might need to show one from our main container of a View, but we could also show a contextualised one next to a button for example.

Finally, if we go back to our initial approach we can update this to something way more elegant:

```swift
@State private var showBanner: Bool = false
@State private var countTap: Int = 0

var body: some View {
        Text(“Some text”)
            .banner(isPresented: $showBanner, data: BannerData(title: “View modifier war”, actionTitle: “NICE”, level: .warning), action: {
                self.countTap += 1
            })
}
```

## Conclusion

Looking to what we have done, we have the ability from any feature/content to access to a nice and consice API that is highly re-usable.

It’s following SwiftUI principles using the tools we have been given. With `ViewModifier` we are also leveraging opaque return type, returning `some View` **we hide the concrete implementation of a `BannerView`** to allow us to update it without having changes on the feature side. That’s a huge benefit.

This is just an example of how we could use `ViewModifier` which can be handy and could be applied in many more places! If you find yourself repeating common layout in multiple places, use a function builder, extract it to its own view or use ViewModifier and have an extension on View.


### Project on GitHub

Everything is available under a project here: 
https://github.com/thomas-sivilay/blog-notification-banner-swiftui
