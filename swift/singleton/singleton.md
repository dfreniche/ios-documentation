theme: Plain jane, 3
autoscale: true
footer: (c) Diego Freniche / @dfreniche / www.freniche.com

# Singletons in Swift

---

## Simple singleton

- no need for GCD
- static vars / constants are guaranteed to be created on first access, thread safe


```swift
class Singleton {
    static let sharedInstance = Singleton()
}
```

---

## Advanced singleton


```swift
class Singleton {
    static let sharedInstance: Singleton = {
        let instance = Singleton()
        // setup code
        return instance
    }()
}
```

---

## Reference

https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/AdoptingCocoaDesignPatterns.html