# Tokikake

Tokikake is a [JQuery.Deferred](http://api.jquery.com/category/deferred-object/)-like API written in Swift.

## Requirements

- iOS 7.0+
- Xcode 6.1

## Usage

### Basic

- `promise.done` is called by `deferred.resolve`.
- `promise.fail` is called by `deferred.reject`.

```swift
let deferred = Deferred<String, String, Float>()
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
    deferred.resolve("ok")
//    deferred.reject("ng")
}

deferred.promise
    .done { (value: String) in
      // do something
    }
    .fail { (error: String) in
      // do something
    }
```


- `promise.then` is called by `deferred.resolve` or `deferred.reject`.
- `promise.always` is always called.

```swift
let deferred = Deferred<String, String, Float>()
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
//    deferred.resolve("ok")
    deferred.reject("ng")
}

deferred.promise
    .then { (value: String?, error: String?) in
      // do something
    }
    .always {
      // do something
    }
```


### Progress

- `promise.progress` is called by `deferred.notify`

```swift
let deferred = Deferred<String, String, Int>()
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
    for i in 0..<10 {
        deferred.notify(i)
    }
    deferred.resolve("ok")
}

deferred.promise
    .progress { (progress: Int) in
        // do something
    }
    .done { (value: String) in
        // do something
    }
```


### Chain

It is possible to return the new promise in `then`, so that you can be chained to the next operation.

```swift
let deferred = Deferred<String, String, Float>()
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
    deferred.resolve("ok")
}

deferred.promise
    .then { (value: String?, error: String?) -> Promise<Int, Int, Float> in
        if error != nil {
            return Deferred<Int, Int, Float>().reject(999).promise
        }
        
        let deferred2 = Deferred<Int, Int, Float>()
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
            deferred2.resolve(1)
        }
        return deferred2.promise
    }
    .done { (value: Int) in
      // do something
    }
    .fail { (error: Int) in
      // do something
    }
    .always {
      // do something
    }
```


### Concurrency

Using `when`, it is possible to wait for the multiple promise completion.

```swift
let deferred1 = Deferred<String, String, Float>()
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
    deferred1.resolve("ok1")
}

let deferred2 = Deferred<String, String, Float>()
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0)) {
    deferred2.resolve("ok2")
}

Promise.when(deferred1.promise, deferred2.promise)
    .progress { (count: Int, total: Int) in
    }
    .done { (values: [String]) in
    }
    .fail { (error: String) in
    }
    .always {
    }
```

## Examples

Other examples habe been described in  [TokikakeTests.swift](https://github.com/yukiasai/Tokikake/blob/master/TokikakeTests/BasicTests.swift)
