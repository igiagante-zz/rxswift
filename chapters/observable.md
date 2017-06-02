### Observer :: Observable <-> Iterator :: Secuence

```swift

class Observable<E> {
  
  typealias Observer<E> = (E) -> Void
  
  var values: [E]
  
  init(values: [E]) {
    self.values = values
  }
  
  func subscribe(_ observer: Observer<E>) {
    for v in values {
      observer(v)
    }
  }
}

let o = Observable<Int>(values: [1,2,3])

o.subscribe { e in
  print(e)
}

// prints
// 1
// 2
// 3

```

For asynchronous code, it keeps reference of all oberservers in order to notify about a new value

```swift
class Observable<E> {
  
  typealias Observer<E> = (E) -> Void
  
  var values: [E]
  var observers: [Observer<E>] = []
  
  init(values: [E]) {
    self.values = values
  }
  
  func subscribe(_ observer: @escaping Observer<E>) {
    observers.append(observer)
    for v in values {
      observer(v)
    }
  }
  
  func append(_ newElement: E ) {
    values.append(newElement)
    for o in observers {
      o(newElement)
    }
  }
}

let o = Observable<Int>(values: [1,2,3])

o.subscribe { e in
  print(e)
}

o.append(4)

// prints
// 1
// 2
// 3
// 4

```

Let's make our Observable reactive to its observer

```swift
class Observable<E> {
  
  typealias Observer<E> = (E) -> Void
  
  typealias SubscriptionHandler = (@escaping Observer<E>) -> Void
  
  let handler: SubscriptionHandler
  
  init(subscriptionHandler: @escaping SubscriptionHandler) {
    self.handler = subscriptionHandler
  }
  
  func subscribe(_ observer: @escaping Observer<E>) {
    self.handler(observer)
  }
}

let o = Observable<Int> { obs in
  obs(1)
  obs(2)
  obs(3)
}

o.subscribe { e in
  print(e)
}

// prints
// 1
// 2
// 3
```

```swift
func tuple() -> Observable<(Int, String)> {
  return Observable<(Int, String)> { obs in // ((Int, String) -> Void)
    obs((1, "one"))
    obs((2, "two"))
  }
}

tuple().subscribe { print($0) }

// prints
// (1, "one")
// (2, "two")

```

It's possible to create more Observable using other Observables

```swift
extension Observable where E == Int {
  func plusOne() -> Observable<Int> {
    return Observable<Int> { obs in
      self.subscribe({ elem in
        obs(elem + 1)
      })
    }
  }
}

let o = Observable<Int> { obs in
  obs(1)
  obs(2)
  obs(3)
}

o.plusOne().subscribe({ print($0) })

// prints
// 2
// 3
// 4
```

And of course create operators, one of the most powerful thing of FRP (Functional Reactive Programming)

```swift
extension Observable {
  func map<U>(_ f: @escaping (E) -> U) -> Observable<U> {
    return Observable<U> { obs in
      self.subscribe({ elem in
        obs(f(elem))
      })
    }
  }
}

let o = Observable<Int> { obs in
  obs(1)
  obs(2)
  obs(3)
}

o.map({ $0 * 2 }).subscribe({ print($0) })

// prints
// 2
// 4
// 6
```

Sometimes observables & observers can use a lot of memory. So, it needs to create a kind of **Unsubscription**

```swift
class Observable<E> {

  typealias Disposable = () -> Void
  
  typealias Observer<E> = (E) -> Void
  
  typealias SubscriptionHandler = (@escaping Observer<E>) -> Disposable
  
  let handler: SubscriptionHandler
  
  init(subscriptionHandler: @escaping SubscriptionHandler) {
    self.handler = subscriptionHandler
  }
  
  func subscribe(_ observer: @escaping Observer<E>) -> Disposable {
    self.handler(observer)
  }
}
```




