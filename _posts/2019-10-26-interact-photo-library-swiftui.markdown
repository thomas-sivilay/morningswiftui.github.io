---
layout: post
title:  "Interact with the photo library with SwiftUI"
date:   2019-10-26 17:38:05 +1000
categories: swiftui
tags: uikit photo-library uiimagepickercontroller bridge
redirect_from: "/blog/interact-photo-library-swiftui"
---
Let’s see how we can bridge SwiftUI and UIKit to allow ourself to use `UIImagePickerController` which gives us the ability to let the user select a photo from his photo library.

![header]({{ site.baseurl }}/assets/images/20191026-picker.gif)

This is a common problem with the first version of SwiftUI that we have today, we would still have to request and use some API that are **only available in UIKit**. We already saw how to do similar things to use `MapKit`, Apple have a tutorial to interface with `UIPageViewController` but here we will only focus on `UIImagePickerController`.

## Introduce `UIViewControllerRepresentable`

First, you need to create a new struct that will conform to `UIViewControllerRepresentable` which is the protocol to implement when we need to interface/integrate with a UIViewController from UIKit.

```swift
struct ImagePickerViewController: UIViewControllerRepresentable {

}
```

This protocol require us to provide:

- A function `makeUIViewController(context:)` to initialize our UIImagePickerController instance. It’s called once when needed to be displayed.
- A function `updateUIViewcontroller(_: context:)` to update the instance of UIImagePickerController which we don’t need in this example.
- A nested `class Coordinator` to communicate with UIKit, in this example it will be the one conforming to `UIImagePickerControllerDelegate`.

The protocol also have a default implementation of
- A function `makeCoordinator()` that we will override to provide our custom nested Coordinator.

## Initialization of the `ImagePicker`

```swift
struct ImagePickerViewController: UIViewControllerRepresentable {

        @Binding var presentationMode: PresentationMode
        @Binding var image: UIImage?

        func makeUIViewController(context: UIViewControllerRepresentableContext<ImagePickerViewController>) -> UIImagePickerController {
            let imagePicker = UIImagePickerController()
            imagePicker.sourceType = UIImagePickerController.SourceType.photoLibrary
            imagePicker.allowsEditing = false
            imagePicker.delegate = context.coordinator
            return imagePicker
        }

        func updateUIViewcontroller(_ uiViewController: UIImagePickerController, context: UIViewControllerRepresentableContext<ImagePickerViewController>)
```

Here we just conform to the protocol by implementing the required methods. We also added a `@Binding var presentedMode: PresentationMode` to let us dismiss this ViewController when user has selected an image. That’s also why we have an optional binding to a UIImage that represent the selected image.

## A Coordinator to implement `UIImagePickerControllerDelegate`

```swift
// Override the default implementation to use our Coordinator
    func makeCoordinator() -> Coordinator {
        return Coordinator(self) // Inject ImagePickerViewController in the Coordinator
    }

    // Nested class inside ImagePickerViewController
    class Coordinator: NSObject: UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        var parent: ImagePickerViewController

        init(_ parent: ImagePickerViewController) {
            self.parent = parent
        }

        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            let imagePicked = info[.originalImage] as! UIImage
            parent.image = imagePicked
            parent.presentationMode.dismiss()
            picker.dismiss(animated: true, completion: nil)
        }

        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.presentationMode.dismiss()
            picker.dismiss(animated: true, completion: nil)
        }
    }
```

If you have already use `UIImagePickerControllerDelegate` there’s nothing really surprising, we have a delegate with a method to handle a success (when user did select) and another method to handle a cancel. In both situation, we dismiss both the UIImagePickerController itself along with our `ImagePickerController.presentationMode`.

In case of success, we assign to the ImagePickerController the image picked by the user.

## Wrap it in a `View`

We are already almost done here but in order to use the `ImagePickerViewController` we need to wrap everything in a view so that any view from SwiftUI can use the picker immediately and easily.

```swift
struct ImagePicker : View {
    @EnvironmentObject var userData: UserData
    @Environment(\.presentationMode) var presentationMode

    var body: some View {
        ImagePickerViewController(image: $userData.image, presentationMode: presentationMode)
    }
}


final class UserData: ObservableObject {
    @Published var image: UIImage? = nil
}

struct ContentView: View {
    @EnvironmentObject var userData: UserData
    @State var pickerIsActive: Bool = false

    var body: some View {
        NavigationView {
            if userData.image != nil {
                Image(uiImage: userData.image!)
            }
            Button(action: {
                self.pickerIsActive = true
            }) {
                Text(“Import image”)
            }
            .sheet(isPresented: $pickerIsActive) {
                ImagePicker().environmentObject(self.userData)
            }
        }
    }
}
```

In that example, we use the `ImagePicker` as the content of a `sheet()` which will show the picker in the new iOS13 card modal presentation. And we keep track on the state with pickerIsActive.
