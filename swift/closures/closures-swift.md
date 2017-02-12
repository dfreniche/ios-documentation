theme: Plain jane, 3
autoscale: true
footer: (c) Diego Freniche / @dfreniche / www.freniche.com

# Swift Closures

---

## Closures or Blocks?

- Blocks -> C (Objective-C)
- Closures -> Swift

---

## Objective-C Blocks

- C-level construct
- Objective-C must be compatible with C, C++

```objectivec
^{
    // code
}
```

---

## Objective-C Blocks

Syntax: not easy to remember

[http://fuckingblocksyntax.com/](http://fuckingblocksyntax.com/) exists for a reason

---

## What is a closure?

A chunk of code. With or without name. That can _closure_ over constants & variables in the context where it is defined

A closure captures the lexical scope where it is defined.

---

## Closures in Swift I: global functions

```Swift
/*: Closure 1: a global function, i.e. a function that's outside any class */

let n = 10

func globalFunction() {
    print("Hello, I'm a global function, capturing n \(n)")
}

globalFunction() // --> Hello, I'm a global function, capturing n 10
```
- `globalFunction` captures `n`

---

## Closures in Swift II: nested functions

```Swift
/* Closure 2: a function inside a function */

func ripley() {
    let name = "Ripley"
    
    func alien() {
        print("Hello, I'm an Alien, I'm going to kill \(name)")
    }
    
    alien()
}

ripley()
```

---

## Closures in Swift III

```Swift
/* Closure 3: a good ol' C block of code */

print("outside block")
    
let block = {
   print("inside block")
}

block()
```

---

## Closures in Swift III

We can't do:

```Swift
print("outside block")
    
{
   print("inside block")
}
```

because the compiler thinks we're expressing a _trailing closure_ (more on that later)

```Swift
print("outside block") {
   print("inside block")
}
```

---

## Closures in Swift III

We can do:

```Swift
print("outside block")
    
_ = {
   print("inside block")
}()
```

- we discard the returned value from the closure

---

## Closures in Swift III

```Swift
let x: ()->() = {   // type of the closure is ()->()
    print("inside block x")
}

x()                 // run the closure

let y: Void = {
    print("inside block y")
}()                 // define & runs the closure. 
                    // As returns nothing, type is Void
```

---

## Why closures?

```swift
let add : (Int, Int) -> Int = { (a, b) in
    a + b
}

let substract: (Int, Int) -> Int = { (a, b) in
    a - b
}

let times: (Int, Int) -> Int = { (a, b) in
    a * b
}

let divide: (Int, Int) -> Int = { (a, b) in
    a / b
}

typealias operation = (Int, Int) -> Int

func doCalculation(a: Int, b: Int, op: operation) -> Int {
    return op(a, b)
}

doCalculation(a: 10, b: 10, op: add)
doCalculation(a: 10, b: 10, op: times)
doCalculation(a: 10, b: 10, op: divide)
```

---


## Closure Expression Syntax

Closure expression syntax has the following general form:

```
{ (`parameters`) -> `return type` in
    `statements`
}
```

---

## Closures: a function returning a function

```swift
import Foundation

let makeCounter: () -> () -> Int = {
    var n = 0
    
    let f = { () -> Int in 
        n = n + 1
        return n
    }
    
    return f
}

makeCounter()()
makeCounter()()

```