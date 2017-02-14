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

## Closures: in Swift, 1st class citizens

- they're _functional_, but 1st class citizens in an OO world
- can be stored in constants / variables 
- can be passed as arguments

---

## Closure Expression Syntax

Closure expression syntax has the following general form:

```
{ (`parameters`) -> `return type` in
    `statements`
}
```

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
block() // call it again!
```

---

## Closures in Swift III

We can't do a 'regular C block':

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

- we run the closure
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




## Trailing closures

```swift
func takeAClosure(closure: () -> ()) {
    closure()
}

takeAClosure(closure: {
    print("Closure inside function call")
})
        
takeAClosure {
    print("Trailing closure!")
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

let counter1 = makeCounter()
let counter2 = makeCounter()

counter1()
counter1()
counter1()

counter2()

```

--- 

## Don't use enumerateObjects to traverse an Array

```swift
var array: NSArray = ["Some", "strings", "in", "an", "array"]

array.enumerateObjects({ (s, i, end) in
    print("i \( i ) s  \( s )")
})
```

---


## High order functions

- Functions that take functions as parameters
- Map, Filter, Reduce, Flatmap
- _Let's do something with these fruits_

```swift
let basket = ["🍏", "🍐", "🍌", "🍉", "🍓"]
```

---

## Map

- "Do *THIS* to every single element in the list"
    - __add 1__ to every element in __this list__
    - __takes__ a function (add one) & a list
    - __returns__ a list
- _with a basket of fruit: peel every fruit in this basket_
- think of __SQL update__  

---

## Map example

```swift
let basket = ["🍏", "🍐", "🍌", "🍉", "🍓"]

basket.map { (e: String) -> String in
    return "🍴" + e
}


```

---

## Map Benefits

- map is a level of indirection
- can be run in parallel (applying a function on element n doesn't depend on element n+1)

---

## Filter

- "I only want oranges from that basket"
- SQL select WHERE clause

---


## Filter example

- "I only want Apples"

```swift

let basket = ["🍏", "🍐", "🍌", "🍉", "🍓", "🖥"]

let newBasket = basket.filter { $0 == "🍏" || $0 == "🖥"
}

```

---

## Reduce

- __takes__ a function (add) & a list
- returns just __one__ element
- _make multi-fruit juice_
- think of AVG function in SQL


---

## Reduce example

```swift

let numbers = [1, 2, 3, 4, 5, 6]

numbers.reduce(0, combine: +) --> 21

```


---



## Flatmap

- Like map, but also "flattens" the list
- _some of the fruits in the basket are wrapped with paper_

---

## Flatmap example

```swift

// how do you add 2 to all the numbers in this array?

let fa2 = [[1,2],[3],[4,5,6]]

let fa2m = fa2.flatMap({$0}).map({$0 + 2})
fa2m


```


---

## Memory management in closures

```swift
class ViewController: UIViewController {
    // ...
    
    var closure: () -> () = {}
    
    @IBAction func clickBtn(_ sender: Any) {
        closure = {
            self.doSomething()   // Error: Call to method 'doSomething' in closure 
                                 // requires explicit 'self.' to make capture semantics explicit 
        }
        
        closure()
    }
   
    func doSomething() {
        print("Hello")
    }
}

```


---


```swift
var closure: () -> () = {}
    
@IBAction func clickBtn(_ sender: Any) {
    closure = { [weak self] in
        self?.doSomething()
    }
    
    closure()
}

func doSomething() {
    print("Hello")
}
```