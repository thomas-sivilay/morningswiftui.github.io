---
layout: post
title:  "ViewState in SwiftUI using ViewBuilder"
date:   2020-01-12 17:38:05 +1000
categories: swiftui
tags: swiftui, viewbuilder, viewstate, ipad, iphone
---
Recently I wanted to **drive a SwiftUI view content based on a `ViewState`**, it became pretty common to use an `Enum` to represent the different state of a View.

```swift
enum ViewState<T> {
    case loading
    case loaded(result: T)
}
```

To keep it simple, we only have two states, either `.loaded` with a result using a generic type or `.loading`. Right now, it’s a bit over-engineered and we could get away with just a boolean. But with this approach I found it useful to be able to later add more states like an error state or an empty state.

## How to use the ViewState?

I already spoiled the final solution, but let’s see different way of doing it.

```swift
// It works but not elegant
struct RestaurantListView: View {
    @ObservedObject var viewModel: RestaurantListViewModel = RestaurantListViewModel()

    var body: some View {
        switch viewModel.viewState {
            case .loading:
                // show a loading indicator, it could be using UIKit.ActivityIndicatorView (check UIViewRepresentable).
                return AnyView(ActivityIndicator())
            case let .loaded(result: restaurants):
                return AnyView(
                    List {
                        ForEach(result) { restaurant in
                            RestaurantListRow(restaurant: restaurant)
                        }
                    }
                )
        }
    }
}
```

From a simple SwiftUI View, we could use directly the `ViewState` and with some control statements achieve our goal. We switch between the different ViewState (here it’s a property `@Published var viewState` in our `ViewModel` but you could also store it as a property @State of the RestaurantListView) and return a different view for each case.

**So what’s wrong with this?** First, since we are using this switch statement, the compiler is going to be annoying and can’t infer the return type so we have to wrap the returned View using AnyView. Second, we only have one example here but if you start to use this ViewState in multiple places of your codebase, chances are you will copy this pattern of the View in all the places your use a ViewState.

## Introducing `ViewBuilder`

With ViewBuilder we can refactor our RestaurantListView to this:

```swift
struct RestaurantListView: View {
    @ObservedObject var viewModel: RestaurantListViewModel = RestaurantListViewModel()

    var body: some View {
        ViewStateView(
            viewState: viewModel.viewState,
            content: { result in
                List {
                    ForEach(result) { restaurant in
                        RestaurantListRow(restaurant: restaurant)
                    }
                }
            }
        )
    }
}
```

Now we only care about how to represent the content when it’s loaded, which also give us the associated value from `ViewState.loaded(result: T)`. Yes we could go further and move the List and ForEach logic in the ViewBuilder by checking if the associated value is an array, but I will leave it up to you since not all array might be represented the same way.

For the other case, when it’s `.loading` since we don’t do anything special here, it’s already treated by the `ViewBuilder` which is named ViewStateView (*I wasn’t inspired*).

This isn’t the first `ViewBuilder` you see, if you already play a bit with SwiftUI, I bet you already used it before. `VStack` is one of the many example. Try to jump to the definition of `List`, `NavigationLink`.. **all those `View` we get out of the boxes is using `ViewBuilder` and we are simply creating our own.**

## Creating a `ViewBuilder`

How to build this `ViewStateView` is straightforward. It is almost like building a View because `ViewBuilder` are Views, the additions is that it allows the caller to inject in a closure form some content that will be used.

```swift
public struct ViewStateView<Content: View, T>: View {

    let content: (T) -> Content
    let viewState: ViewState<T>

    public init(viewState: ViewState<T>, @ViewBuilder: content: @escaping (T) -> Content) {
        self.content = content()
        self.viewState = viewState
    }

    var body: some View {
        switch viewState {
            case .loaded(let result):
                return AnyView(content(result))
            case .loading:
                return AnyView(LoadingView(style: .large))
        }
    }
}

struct LoadingView: UIViewRepresentable {

    let style: UIActivityIndicatorView.Style

    func makeUIView(context: UIViewRepresentableContext<ActivityIndicator>) -> UIActivityIndicatorView {
        return UIActivityIndicatorView(style: style)
    }

    func updateUIView(_ uiView: UIActivityIndicatorView, context: UIViewRepresentableContext<ActivityIndicator>) {
        uiView.startAnimating()
    }
}
```

## Conclusion

This was a really simple example of using `@ViewBuilder` and solving our initial goal to represent `ViewState<T>` in SwiftUI but there’s many other example where you could apply this again.

### Bonus: Show a VStack on iPhone but switch to HStack on iPad

One places where I actually used it before was to handle switching between a Vertical and Horizontal stack. The rules is arbitrary, but I ended up re-using this alternative Stack in multiple places in another project:

```swift
struct Stack<Content: View> {

    var content: Content

    init(@ViewBuilder content: @escaping () -> Content) {
        self.content = content()
    }

    var body: some View {
        if AppEnvironment.current.isIphone {
            return AnyView(VStack(content))
        } else {
            return AnyView(HStack(content))
        }
    }
}
```
