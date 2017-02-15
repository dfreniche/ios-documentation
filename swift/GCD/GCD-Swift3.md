theme: Plain jane, 3
autoscale: true
footer: (c) Diego Freniche / @dfreniche / www.freniche.com

# GCD: Grand Central Dispatch & Swift 3

---

## GCD

- introduced in iOS 4
- it's a C library
- real name: `libdispatch` from Apple [^1]


[^1]: https://github.com/apple/swift-corelibs-libdispatch

---

## Thread basics (for debug purposes)


```swift
// To check we're on main Thread

assert(Thread.current == Thread.main)

// To debug thread

print(Thread.current.description)

// To delay a thread

Thread.sleep(forTimeInterval: 1000)

```

---

## GCD queues

- organized around the concept of __queues__
- no need to create threads manually
- you send tasks (some code) to the queue and GCD creates threads for you
- serial and concurrent queues

```
1. create (or get) a queue
2. dispatch some code (TASK1)
3. dispatch some more code (TASK2)
4. the queue runs that code
```

---

## GCD queues

- Main queue: runs on the __main thread__ and is a __serial queue__
- Global queues: __concurrent queues__ that are shared by the whole system
- Custom queues: queues that you create which can be serial or concurrent



---


## Good ol' GCD

- Objective-C + Pyramid of Doom 😱

```objectivec
dispatch_queue_t serialQueue = dispatch_queue_create("com.unique.name.queue", DISPATCH_QUEUE_SERIAL);

dispatch_async(serialQueue, ^{
    [self ReadAllImagesFromPhotosLibrary];
    dispatch_async(serialQueue, ^{
        [self WriteFewImagestoDirectory];
        dispatch_async(serialQueue, ^{
            [self GettingBackAllImagesFromFolder]; 
            dispatch_async(serialQueue, ^{
                [self MoveToNextView];
            });
        });
    });
}); 
```

---

## In "Old" swift

```swift
let concurrentQueue = dispatch_queue_create("com.swift3.imageQueue", DISPATCH_QUEUE_CONCURRENT)
```

This doesn't look like Swift, still looks like C

---

## Create a custom serial queue

- Serial queue: we can dispatch closures with code to the serial queue
- closures get executed one after anoother __in order__
- until task1 doesn't finish, we can't start task2
- we use `DispatchQueue` to create new GCD queues


```swift
let serialQueue = DispatchQueue(label: "queuename")
serialQueue.sync { 

}
```

---

## Serial queue: dispatch_sync

```swift

let serialQueue = DispatchQueue(label: "serial")
        
print("Start")              // prints

// sync blocks this thread until the closure is done
serialQueue.sync {          // add this to the serial queue, executes and waits
    for i in 0...10 {
        print("*** \( i )")
    }
}                           // when finished we continue

print("Middle")             // prints

serialQueue.sync {          // add this to the serial queue, executes and waits
    for i in 0...10 {
        print("--- \( i )")
    }
}

print("End")                // prints
```

---


## Serial queue: dispatch_async

```swift

let serialQueue = DispatchQueue(label: "serial")
        
print("Start")              // prints

// async don't block this thread, just dispatches
serialQueue.async {          // dispatch this to the serial queue, then CONTINUES
    for i in 0...10 {
        print("*** \( i )")
    }
}                           

print("Middle")             // prints this before previous block has finished

serialQueue.sync {          // dispatch this to the serial queue, then CONTINUES
    for i in 0...10 {
        print("--- \( i )")
    }
}

print("End")                // prints
```

---

## Creating a custom concurrent queue

- Concurrent queue: we can dispatch closures with code to the queue
- all closures get executed __as they enter the queue__
- task1 doesn't need to finish to start task2
- task1 & task2 can be running __at the same time__
- tasks are guaranteed to start in the order they were added to the queue


```swift
let backgroundQueue = DispatchQueue(label: "com.example.my-concurrent-queue",
                                            attributes: DispatchQueue.Attributes.concurrent)

// dispatches this block and continues
backgroundQueue.async {
    for i in 0...100 {
        print("*** \( i )")
    }
}

print("Middle") // prints from the main thread

// dispatches this block and continues. This block can run at the same time than the 1st
backgroundQueue.async {
    for i in 0...100 {
        print("--- \( i )")
    }
}

```

---

## Get main queue

```swift

// async

DispatchQueue.main.async {

}

// synchronously

DispatchQueue.main.sync {

}

```

---

## Running something after a delay

```swift
DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) { // in half a second...
    print("Are we there yet?")
}
```

---


## To get one of the global background threads

- sometimes we just want to launch something in backgorund
- not interested in create a custom queue
- global queues: __concurrent queues__ that are shared by the whole system

```swift
DispatchQueue.global(qos: .userInitiated).async {
    print("user initiated task")
}

// long version
DispatchQueue.global(qos: DispatchQoS.QoSClass.background).async {
    print("user initiated task")   
}

// short version
DispatchQueue.global().async {
    // qos' default value is ´DispatchQoS.QoSClass.default`
}

```


---

## Quality of service

Quality of Service (QoS) specifiers were introduced in OS X 10.10 / iOS 8.0, providing a clearer way for the system to prioritize work and deprecating the old priority specifiers.

- User-interactive: Work is virtually instantaneous.
- User-initiated: Work is nearly instantaneous, such as a few seconds or less.
- Utility: Work takes significant time, such as minutes or hours.
- Default: The priority level of this QoS falls between user-initiated and utility. This QoS is not intended to be used by developers to classify work. Work that has no QoS information assigned is treated as default, and the GCD global queue runs at this level.

---

## DispatchWorkItem

You can also use same block of code many times by wrapping it in DispatchWorkItem instance and passing it to DispatchQueue

```swift

let block = DispatchWorkItem {
    print("do something")
}
DispatchQueue.main.async(execute: block)
```


---

## Semaphores

```swift
func synchronized( lock:AnyObject, block:() throws -> Void ) rethrows
{
    objc_sync_enter(lock)
    defer {
        objc_sync_exit(lock)
    }

    try block()
}


```

---

## Reference:

- http://stackoverflow.com/a/37806522/225503
- http://stackoverflow.com/a/37801371/225503
- https://swiftable.io/2016/06/dispatch-queues-swift-3/
- https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html
- https://www.raywenderlich.com/148513/grand-central-dispatch-tutorial-swift-3-part-1