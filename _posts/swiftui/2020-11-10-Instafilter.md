---
layout: post
title: Instafilter
description: Learn to link SwiftUI, UIKit, and Core Image in one app
summary: Learn to link SwiftUI, UIKit, and Core Image in one app
tags: [swiftui, hackingwithswift]
permalink: "/swiftui/Instafilter"
---

### Keywords

- UIKit
- Core Image
- UIImage, CGImage, CIImage
- ActionSheet
- UIImagePickerController
- Custom binding

### Keynotes

1. `@objc` tells Swift to create a method that can be read by Objective-C
2. We can place optional views directly into a SwiftUI view hierarchy. SwiftUI will only render them if they have a value
3. When creating a custom Binding, we must specify both `get` and `set` closures
4. Calling `UIImageWriteToSavedPhotosAlbum()` will fail if the user denied access to their photo library
5. We cannot detect when an `@State` property changes using a property observer. We need to use a custom binding instead
6. Core Image filters take different input keys. Trying to provide a key that isn't supported will cause a crash
7. To make a SwiftUI view wrap a UIKit view controller, we must make it confrom to `UIViewControllerRepresentalbe` (which already conforms to `View`)
8. `Coordinators` act as bridges between SwiftUI's views and UIKit's view controllers
9. The Binding struct is generic. This means we don't create bindings without further context, but instead say "this is a binding for a string"
10. Core Image lets us apply graphical filters to an image.
It performs the transformation in the GPU, which is what makes it so fast on devices

### Code

```swift
//
//  ContentView.swift
//  Instafilter
//

import SwiftUI
import CoreImage
import CoreImage.CIFilterBuiltins

struct ContentView: View {
    @State private var image: Image?
    @State private var filterIntensity = 0.5
    @State private var showingImagePicker = false
    @State private var inputImage: UIImage?
    @State private var currentFilter: CIFilter = CIFilter.sepiaTone()
    @State private var showingFilterSheet = false
    @State private var processedImage: UIImage?
    
    let context = CIContext()
    
    var body: some View {
        let intensity = Binding<Double>(
            get: {
                self.filterIntensity
            },
            set: {
                self.filterIntensity = $0
                self.applyProcessing()
            }
        )
        
        return NavigationView {
            VStack {
                ZStack {
                    Rectangle()
                        .fill(Color.secondary)
                    
                    /// Display the image.
                    if let image = image {
                        image
                            .resizable()
                            .scaledToFit()
                    } else {
                        Text("Tap to select a picture")
                            .foregroundColor(.white)
                            .font(.headline)
                    }
                }
                .onTapGesture(count: 1, perform: {
                    /// Select an image.
                    self.showingImagePicker = true
                })
                
                HStack {
                    Text("Intensity")
                    Slider(value: intensity)
                }
                .padding(.vertical)
                
                HStack {
                    Button("Change Filter") {
                        /// Change filter.
                        self.showingFilterSheet = true
                    }
                    
                    Spacer()
                    
                    Button("Save") {
                        /// Save the picture.
                        guard let processedImage = self.processedImage else { return }
                        
                        let imageSaver = ImageSaver()
                        imageSaver.successHandler = {
                            print("Success!")
                        }
                        imageSaver.errorHandler = {
                            print("Oops: \($0.localizedDescription)")
                        }
                        imageSaver.writeToPhotoAlbum(image: processedImage)
                    }
                }
            }
            .padding([.horizontal, .bottom])
            .navigationBarTitle("Instafilter")
            .sheet(isPresented: $showingImagePicker, onDismiss: loadImage, content: {
                ImagePicker(image: self.$inputImage)
            })
            .actionSheet(isPresented: $showingFilterSheet, content: {
                ActionSheet(title: Text("Select a filter"), buttons: [
                    .default(Text("Crystallize")) { self.setFilter(CIFilter.crystallize()) },
                    .default(Text("Edges")) { self.setFilter(CIFilter.edges()) },
                    .default(Text("Gaussian Blur")) { self.setFilter(CIFilter.gaussianBlur()) },
                    .default(Text("Pixellate")) { self.setFilter(CIFilter.pixellate()) },
                    .default(Text("Sepia Tone")) { self.setFilter(CIFilter.sepiaTone()) },
                    .default(Text("Unsharp Mask")) { self.setFilter(CIFilter.unsharpMask()) },
                    .default(Text("Vignette")) { self.setFilter(CIFilter.vignette()) },
                    .cancel()
                ])
            })
        }
    }
    
    func applyProcessing() {
        let inputKeys = currentFilter.inputKeys
        if inputKeys.contains(kCIInputIntensityKey) { currentFilter.setValue(filterIntensity, forKey: kCIInputIntensityKey) }
        if inputKeys.contains(kCIInputRadiusKey) { currentFilter.setValue(filterIntensity * 200, forKey: kCIInputRadiusKey) }
        if inputKeys.contains(kCIInputScaleKey) { currentFilter.setValue(filterIntensity * 10, forKey: kCIInputScaleKey) }
        
        guard let outputImage = currentFilter.outputImage else { return }
        
        if let cgimg = context.createCGImage(outputImage, from: outputImage.extent) {
            let uiImage = UIImage(cgImage: cgimg)
            image = Image(uiImage: uiImage)
            processedImage = uiImage
        }
    }
    
    func loadImage() {
        guard let inputImage = inputImage else { return }
        
        let beginImage = CIImage(image: inputImage)
        currentFilter.setValue(beginImage, forKey: kCIInputImageKey)
        applyProcessing()
    }
    
    func setFilter(_ filter: CIFilter) {
        currentFilter = filter
        loadImage()
    }
}

class ImageSaver: NSObject {
    var successHandler: (() -> Void)?
    var errorHandler: ((Error) -> Void)?
    
    func writeToPhotoAlbum(image: UIImage) {
        UIImageWriteToSavedPhotosAlbum(image, self, #selector(saveError), nil)
    }
    
    @objc func saveError(_ image: UIImage, didFinishSavingWithError error: Error?, contextInfo: UnsafeRawPointer) {
        if let error = error {
            errorHandler?(error)
        } else {
            successHandler?()
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

```swift
//
//  ImagePicker.swift
//  Instafilter
//

import SwiftUI

struct ImagePicker: UIViewControllerRepresentable {
    @Binding var image: UIImage?
    @Environment(\.presentationMode) var presentationMode
    
    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        var parent: ImagePicker
        
        init(_ parent: ImagePicker) {
            self.parent = parent
        }
        
        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let uiImage = info[.originalImage] as? UIImage {
                parent.image = uiImage
            }
            
            parent.presentationMode.wrappedValue.dismiss()
        }
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {
        
    }
    
}
```

