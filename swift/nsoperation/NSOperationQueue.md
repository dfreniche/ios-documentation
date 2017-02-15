theme: Plain jane, 3
autoscale: true
footer: (c) Diego Freniche / @dfreniche / www.freniche.com

# NSOperationQueue

---


- Alternative to using blocks / closures for multithread / background tasks
- uses GCD under the hood
- we can cancel operations running
- we can chain operations

- NSOperation == closure
- NSOperationQueue == GCD queue


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

## Queue: where we run operations

```swift
let queue = OperationQueue()

queue.addOperation {
    print("Hello ")
}

queue.addOperation {
    print("is it me")
}

queue.addOperation {
    print("you're looking for?")
}

```


---

## Operation: the task at hand to do

- extend from `Operation` A.K.A. `NSOperation`

```swift
class MyOp: Operation {
    override func main() {
        print("Operation at work! " + message + Thread.current.description)
    }
}

let op1 = MyOp()
queue.addOperation(op1)

```

---

## Dependencies between operations

```swift

class MyOp: Operation {
    let message: String
    
    init(message: String) {
        self.message = message
    }
    
    override func main() {
        print("Operation at work! " + message)
    }
}


let op1 = MyOp(message: "op1")
let op2 = MyOp(message: "op2")

op2.addDependency(op1)
queue.addOperation(op2)
queue.addOperation(op1)
```

---

## Going back to Main Thread I

- `performSelector`

```swift
class MyOp: Operation {
    let message: String
    
    init(message: String) {
        self.message = message
    }
    
    override func main() {
        print("Operation at work! " + message)
        
        self.performSelector(onMainThread: #selector(MyOp.onMain), with: nil, waitUntilDone: false)
    }
    
    func onMain() {
        print("Main thread")
    }
}

```

---

## Going back to Main Thread II

- `Operation.main`


```swift
class MyOp: Operation {
    let message: String
    
    init(message: String) {
        self.message = message
    }
    
    override func main() {
        print("Operation at work! " + message)
        
        OperationQueue.main.addOperation {
            print("Main thread" + Thread.current.description)
        }
    }
}

```

