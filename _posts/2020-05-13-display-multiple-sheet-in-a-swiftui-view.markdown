---
layout: post
title:  "Display multiple Sheet in a SwiftUI View"
date:   2020-05-13 17:38:05 +1000
categories: swiftui
tags: sheet modal viewmodifier
redirect_from: "display-multiple-sheet-in-a-swiftui-view"
---
SwiftUI views has a **ViewModifier** `.sheet` that allows us to present a modal view based on a state .isPresented which is a Bool. **What happened when we need to present more than one sheet?**

## You can’t use multiple `.sheet` view modifiers

One naive approach could be to structure our view with multiple view modifiers, have multiple `.sheet` with multiple Boolean to drive when and what needs to be displayed.

```swift
struct MyView: View {
    @State var showSheetA: Bool = false
    @State var showSheetB: Bool = false

    var body: some View {
        // This doesn’t work
        Text("My view")
            .sheet(isPresented: $showSheetA, content: { Text("Sheet A") })
            .sheet(isPresented: $showSheetB, content: { Text("Sheet B") })
    }
}
```

With this approach when `showSheetB` is true, we see a modal that display the content `Text("Sheet B")` but when `showSheetA` is true nothing happens. **Only the last `.sheet` view modifier will be used.**

In the case where `showSheetA` and `showSheetB` are both true, we might apply the first modifier but then the second similar view modifier will then replace what has been done previously.

## One sheet, multiple content

One solution is to have only one view modifier but display different content based on our needs.

Again, the simplest approach could be to introduce another state and use more booleans.

```swift
struct MyView: View {
    @State var showSheet: Bool = false
    @State var showA: Bool = false
    @State var showB: Bool = false

    var body: some View {
        // This works but is it elegant?
        Text("My view")
            .sheet(isPresented: $showSheet, content: { self.sheet })
    }

    private var sheet: some View {
        if showA {
            Text("Sheet A")
        } else if showB {
            Text("Sheet B")
        } else {
            // Of course that will never happen...
            EmptyView()
        }
    }
}
```

Since this works, we can try to improve it. **There must be a more elegant and safer approach** that will represent better the state of the view. We probably should avoid all those booleans and find a data structure that represent well our problem.

```swift
final class ActiveSheet: ObservableObject {
    enum Kind {
        case a
        case b
        case none
    }
    @Published var kind: Kind = .none {
        didSet { showSheet = kind != .none }
    }
    @Published var showSheet: Bool = false
}

struct MyView: View {
    @ObservedObject var activeSheet: ActiveSheet = ActiveSheet()

    var body: some View {
        Text("My view")
            .sheet(isPresented: self.$activeSheet.showSheet, content: { self.sheet })
    }

    private var sheet: some View {
        switch activeSheet.kind {
            case .none: return AnyView(EmptyView())
            case .a: return AnyView(Text("Sheet a"))
            case .b: return AnyView(Text("Sheet b"))
        }
    }
}
```

This way, we hold all different kind of sheets in our SwiftUI View. It gives **more clarity and scale better** when we will need to add more kind of sheets.

For example, I had to implement a sheet that will display itself given a URL of a file. All I had to do is whenever the file was ready: `activeSheet.kind = .export(url: url)` and the view modifier that we set up declaratively will now present a new content.

We could probably still improve this, somehow removing the `@Published var showSheet: Bool = false` but that’s it for today.
