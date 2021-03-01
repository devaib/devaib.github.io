---
layout: post
title: Grand Central Dispatch
description: Understand Grand Central Dispatch.
summary: Understand Grand Central Dispatch.
tags: [swift, grandcentraldispatch, dispatchqueue]
permalink: "/swift/GrandCentralDispatch"
---

- [Mastering Grand Central Dispatch - Cocoacasts](https://cocoacasts.com/mastering-grand-central-dispatch-what-is-grand-central-dispatch)
- [Swift clip: Dispatch queues - Swift by Sundell](https://www.swiftbysundell.com/clips/2/)

### Keywords

- Main Dispatch Queue
- Global Dispatch Queue
- Synchronous and Asynchronous Execution
- Serial and Concurrent Dispatch Queue

### What is Grand Central Dispatch

Grand Central Dispatch is a technology developed by Apple to optimize application support for system with multi-core processors and other symmetric multiprocessing system. It is an implementation of task parallelism based on the thread pool pattern and it operates at the system level. Your application operates in a sandbox, which means that it is unaware of other processes running on the system at the same time. Because Grand Central Dispatch operates at the system level, it has an accurate view of the proceses that are runnign and the resources that are available.

GCD was first released with Mac OS X 10.6, and is also available with iOS 4 and above.The name "Grand Central Dispatch" is a reference to Grand Central Terminal.

<center><img src="/assets/img/swift/2021-02-21/sandbox.png" alt="drawing" width="90%"/></center>

### Managing Threads

- Manages a pool of threads
- Decides which thread is used for execution
- Makes no guarantee as to which thread is used for execution (Exception: work submitted to the main dispatch queue is guaranteed to be executed on the main thread)

### Dispatch Queue

Dispatch queue is an object that manages the execution of task serially or concurrently on your app's main thread or on a background thread. An application interacts with Grand Central Dispatch through dispatch queues. A dispatch queue enqueues and dequeues work in FIFO (first in, first out) order.

#### Different Types of Dispatch Queues

```swift
let mainQueue = DispatchQueue.main

let globalQueue = DispatchQueue.global()

let customQueue = DispatchQueue(
    label: "com.myapp.queue"
)

let backgroundQueue = DispatchQueue(
    label: "com.myapp.queue.background",
    qos: .background
)

let concurrentQueue = DispatchQueue(
    lable: "com.myapp.queue.concurrent",
    attributes: .concurrent
)

DispatchQueue.main.async {
    print("Hello world")
}
```

#### Working with Dispatch Queues

```swift
import UIKit

class ViewController: UIViewController {

    // MARK: - Properties
    
    @IBOutlet var imageViews: [UIImageView]!
    
    // MARK: -
    
    private lazy var images: [URL] = [
        URL(string: "https://cdn.cocoacasts.com/7ba5c3e7df669703cd7f0f0d4cefa5e5947126a8/1.jpg")!,
        Bundle.main.url(forResource: "2", withExtension: "jpg")!,
        URL(string: "https://cdn.cocoacasts.com/7ba5c3e7df669703cd7f0f0d4cefa5e5947126a8/3.jpg")!,
        Bundle.main.url(forResource: "4", withExtension: "jpg")!
    ]
    
    // MARK: -
    
    private let dispatchQueue = DispatchQueue(label: "My Dispatch Queue")
    
    // MARK: - View Life Cycle
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        print("Start \(Date())")
        
        for (index, imageView) in imageViews.enumerated() {
            // Fetch URL
            let url = images[index]
            
            dispatchQueue.async { [weak self] in
                // Populate Image View
                self?.loadImage(with: url, for: imageView)
            }
        }
        
        print("Finish \(Date())")
    }
    
    // MARK: - Helper Methods
    
    private func loadImage(with url: URL, for imageView: UIImageView) {
        // Load Data
        guard let data = try? Data(contentsOf: url) else {
            return
        }
        
        // Create Image
        let image = UIImage(data: data)
        
        DispatchQueue.main.async {
            // Update Image View in the main thread (by assigning this to the main dispatch queue)
            imageView.image = image
        }
    }

}

```

### Serial vs Concurrent Dispatch Queue

#### Serial Dispatch Queue

A serial dispatch queue has following features:

- The order in which the blocks of work are executed is predictable
- The order in which the blocks of work finish executing is also predictable

The main dispatch queue is a serial dispatch queue.

<center><img src="/assets/img/swift/2021-02-21/serial_dispatch_queue.png" alt="drawing" width="90%"/></center>

```swift
private let serialDispatchQueue = DispatchQueue(label: "Serial Dispatch Queue")

override func viewDidLoad() {
    super.viewDidLoad()

    print("Start \(Date())")

    for (index, imageView) in imageViews.enumerated() {
        // Fetch URL
        let url = images[index]

        serialDispatchQueue.async { [weak self] in
            // Populate Image View
            self?.loadImage(with: url, for: imageView)
        }
    }

    print("Finish \(Date())")
}
```

#### Concurrent Dispatch Queue

The order of completion is not under control of Grand Central Dispatch in concurrent dispatch queue.

<center><img src="/assets/img/swift/2021-02-21/concurrent_dispatch_queue.png" alt="drawing" width="90%"/></center>

```swift
private let concurrentDispatchQueue = DispatchQueue(label: "Concurrent Dispatch Queue", attribute: .concurrent)

override func viewDidLoad() {
    super.viewDidLoad()

    print("Start \(Date())")

    for (index, imageView) in imageViews.enumerated() {
        // Fetch URL
        let url = images[index]

        concurrentDispatchQueue.async { [weak self] in
            // Populate Image View
            self?.loadImage(with: url, for: imageView)
        }
    }

    print("Finish \(Date())")
}
```

### Global Dispatch Queue

Global dispatch queue are global to the application, which means they can be accessed from anywhere in your application. It can be useful to create a dedicated dispatch queue for a particular task or a group of tasks, such as data synchronization. By using a dedicated dispatch queue, the owner of the dispatch queue can carefully control which objects have access to the dispatch queue.

The `global()` class method defines an optional parameter of type `DispatchQoS.QoSClass`. QoS stands for **quality of service** and the `DispatchQoS` struct encapsulates quality of service classes. The `QoSClass` enum defines a number of quality of service classes.

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    print("Start \(Date())")

    for (index, imageView) in imageViews.enumerated() {
        // Fetch URL
        let url = images[index]

        DispatchQueue.global(qos: .utility).async { [weak self] in
            // Populate Image View
            self?.loadImage(with: url, for: imageView)
        }
    }

    print("Finish \(Date())")
}
```

### Asynchronous vs Synchronous Execution

#### Asynchronous

```swift
import Foundation

let dispatchQueue = DispatchQueue.global()

print("before")

dispatchQueue.async {
    let data = try! Data(contentsOf: URL(string: "https://cdn.cocoacasts.com/7ba5c3e7df669703cd7f0f0d4cefa5e5947126a8/1.jpg")!)
    print(data.count)
}

print("after")
```

```
before
after
2328941
```

The `async(execute:)` method returns immediately. It submits the block of work to the global dispatch queue and returns control to the thread from which the `async(execute:)` method is invoked.

<center><img src="/assets/img/swift/2021-02-21/async_execution.png" alt="drawing" width="90%"/></center>

#### Synchronous

```swift
import Foundation

let dispatchQueue = DispatchQueue.global()

print("before")

dispatchQueue.sync {
    let data = try! Data(contentsOf: URL(string: "https://cdn.cocoacasts.com/7ba5c3e7df669703cd7f0f0d4cefa5e5947126a8/1.jpg")!)
    print(data.count)
}

print("after")
```

```
before
2328941
after
```

The `sync(execute:)` method doesn't immediately return control to the thread from which the `sync(execute:)` method is invoked. In other words, the `sync(execute:)` method blocks the thread from which it is invoked, that is, the calling thread. Control is returned after the block of work has finished executing.

<center><img src="/assets/img/swift/2021-02-21/sync_execution.png" alt="drawing" width="90%"/></center>

