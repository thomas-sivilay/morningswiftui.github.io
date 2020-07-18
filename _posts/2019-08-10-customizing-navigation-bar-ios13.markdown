---
layout: post
title:  "Customising navigation bar in iOS13 with `UINavigationBarAppearance`"
date:   2019-08-10 17:38:05 +1000
categories: ios13
tags: ios13 navigationbar uikit
redirect_from: "/blog/customizing-navigation-bar-ios13"
---

Base on [Modernizing Your UI for iOS 13 - WWDC 2019](https://developer.apple.com/videos/play/wwdc2019/224/) in iOS13 the new way to customise your navigation bar is to use `UINavigationBarAppearance`. It also gives you more granularity over:

- .scrollEdgeAppearance will be used if your view contains a scrollview and it’s scrolled at the top
- .compactAppearance for iPhone in landscape
- .standardAppearance for the rest

## NavigationBar shadow

I’ll suggest to start by defining if you need to show the shadow of the navigation bar or not. Because on iOS13, you use one of these methods which can override your previous customisations.

```swift
func customApperance() {
    let navBarAppearance = UINavigationBarAppearance()
    // Will remove the shadow and set background back to clear
    navBarAppearance.configureWithTransparentBackground()

    navBarAppearance.configureWithOpaqueBackground()

    navBarAppearance.configureWithDefaultBackground()
}
```

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">With the new UINavigationBarAppearance on iOS13 we can customise how the nav bar looks like when it&#39;s scrolled at the top or when user is actually scrolling down! Also, no more shadow (if you remove it) when adding a searchController. <a href="https://twitter.com/hashtag/UINavigationBarAppearance?src=hash&amp;ref_src=twsrc%5Etfw">#UINavigationBarAppearance</a> <a href="https://twitter.com/hashtag/iOS13?src=hash&amp;ref_src=twsrc%5Etfw">#iOS13</a> <a href="https://twitter.com/hashtag/UIKit?src=hash&amp;ref_src=twsrc%5Etfw">#UIKit</a> <a href="https://t.co/QxGT5Lrh3E">pic.twitter.com/QxGT5Lrh3E</a></p>&mdash; Thomas Sivilay (@thomassivilay) <a href="https://twitter.com/thomassivilay/status/1159995320396201985?ref_src=twsrc%5Etfw">August 10, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

One of the cool thing with iOS13 is that now if you have a `searchController` attached to your navigationItem you can also use this navBarAppearance to remove the shadow! Something that wasn’t as easy (or even possible?) for previous iOS versions.

```swift
func customSearchBar() {
        let transparentAppearance = navBarAppearance.copy()
        transparentAppearance.configureWithTransparentBackground()

        navigationItem.searchController = searchController
        searchController.navigationItem.scrollEdgeAppearance = transparentAppearance
    }
```

## NavigationBar background

![Nav 1]({{ site.baseurl }}/assets/images/20190810-1.jpg)

After configuring the appearance, you can now override the background color.

```swift
func customBackground() {
    navBarAppearance.backgroundColor = UIColor.systemGray
}
```
This also works even if you decided previously to call `.configureWithTransparentBackground()`.

## Navigation title

There’s two cases here, either you use a large title or not which results in two different properties to update:

```swift
func customNavBarTitle() {
    navBarAppearance.titleTextAttributes = [
        .foregroundColor : UIColor.purple,
        .font : UIFont.italicSystemFont(ofSize: 17)
    ]

    navBarAppearance.largeTitleTextAttributes = [
        .foregroundColor : UIColor.systemBlue,
    ]
}
```
![Nav 2]({{ site.baseurl }}/assets/images/20190810-2.jpg)

One note here, it’s good to use a fixed font size and not preferred font size. The navigation bar height doesn’t grow and don’t give more space, this is also the default behaviour anyway.

There is also .titlePositionAdjustment that allows you to vertically and horizontally move the title, which could be useful if you use a custom font.

```swift
navBarAppearance.titlePositionAdjustment = UIOffset(horizontal: 0, vertical: 3)
```

## Navigation Items

![Nav 3]({{ site.baseurl }}/assets/images/20190810-3.jpg)

With UINavigationBarAppearance you can also individually customise the navigation bar button items appearance using UIBarButtonItemAppearance.

```swift
func customBarButtonItems() {
    let buttonAppearance = UIBarButtonItemAppearance()
    buttonAppearance.normal.titleTextAttributes = [.foregroundColor : UIColor.darkGray]        
    navBarAppearance.buttonAppearance = buttonAppearance
}
```

There is a granularity for each bar button type:

```swift
navBarAppearance.doneButtonAppearance
navBarAppearance.backButtonAppearance
```

Remember that by default, the done button is bolded to emphasize on the main action to take!

## Everything together

```swift
func customizeNavigationBar() {
    navigationItem.setLeftBarButton(.init(barButtonSystemItem: .cancel, target: nil, action: nil), animated: true)
    navigationItem.setRightBarButton(.init(barButtonSystemItem: .done, target: nil, action: nil), animated: true)

    navigationItem.searchController = searchController

    let navBarAppearance = UINavigationBarAppearance()

    // Call this first otherwise it will override your customizations
    navBarAppearance.configureWithDefaultBackground()

    let buttonAppearance = UIBarButtonItemAppearance()
    let titleTextAttributes = [NSAttributedString.Key.foregroundColor : UIColor.systemGray]
    buttonAppearance.normal.titleTextAttributes = titleTextAttributes

    navBarAppearance.titleTextAttributes = [
        .foregroundColor : UIColor.purple, // Navigation bar title color
        .font : UIFont.italicSystemFont(ofSize: 17) // Navigation bar title font
    ]

    navBarAppearance.largeTitleTextAttributes = [
        .foregroundColor : UIColor.systemBlue, // Navigation bar title color
    ]

    // appearance.backgroundColor = UIColor.systemGray // Navigation bar bg color
    // appearance.titlePositionAdjustment = UIOffset(horizontal: 0, vertical: 8) // Only works on non large title

    let transNavBarAppearance = navBarAppearance.copy()
    transNavBarAppearance.configureWithTransparentBackground()
    navigationController?.navigationBar.standardAppearance = navBarAppearance
    navigationController?.navigationBar.compactAppearance = navBarAppearance
    navigationController?.navigationBar.scrollEdgeAppearance = transNavBarAppearance

    searchController.navigationItem.standardAppearance = navBarAppearance
    searchController.navigationItem.compactAppearance = navBarAppearance
    searchController.navigationItem.scrollEdgeAppearance = transNavBarAppearance

    title = "Title"

    navigationController?.navigationBar.prefersLargeTitles = true // Activate large title
}
```
