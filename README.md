## VIPER Architecture

## PromiseKit

**PromiseKit** is a library that helps with asynchronous programming in swift and Objective-C. It is explained as "PromiseKit is a thoughtful and complete implementation of promises" in the official doc.

Advantage of using PromiseKit over traditional iOS async programming is:
* Cleaner and readable code
* Quick learning carve 
* Very easy to call multiple async functions and managing it.

Example: 

```swift
UIApplication.shared.isNetworkActivityIndicatorVisible = true

let fetchImage = URLSession.shared.dataTask(.promise, with: url).compactMap{ UIImage(data: $0.data) }
let fetchLocation = CLLocationManager.requestLocation().lastValue

firstly {
    when(fulfilled: fetchImage, fetchLocation)
}.done { image, location in
    self.imageView.image = image
    self.label.text = "\(location)"
}.ensure {
    UIApplication.shared.isNetworkActivityIndicatorVisible = false
}.catch { error in
    self.show(UIAlertController(for: error), sender: self)
}
```

### Common Syntax: 

**then and done**: then and done are kinda like completion handlers. But the difference is that we can use them to chain multiple async calls or other operators like error handling. It is also great for readablity. 

Example:
```swift
firstly {
    login()
}.then { creds in
    fetch(avatar: creds.user)
}.done { image in
    self.imageView = image
}
```

**catch**: catch is closure for error handling. What is interesting here is that the catch closure catches all the errors of the whole chain. 

Example:
```swift
firstly {
    login()
}.then { creds in
    fetch(avatar: creds.user)
}.done { image in
    self.imageView = image
}.catch {
    // any errors in the whole chain land here
}
```

**ensure**: ensure closure is called no matter the promise succeed or failed. We should must called codes inside ensure clause.

Example: 
```swift
firstly {
    UIApplication.shared.isNetworkActivityIndicatorVisible = true
    return login()
}.then {
    fetch(avatar: $0.user)
}.done {
    self.imageView = $0
}.ensure {
    UIApplication.shared.isNetworkActivityIndicatorVisible = false
}.catch {
    //…
}
```

**when**: parallel multiple async call is so much easier with promise. when takes multiple promises and wait for them to end and return a promise containing the results.

Example:
```swift
firstly {
    when(fulfilled: operation1(), operation2())
}.done { result1, result2 in
    //…
}
```

**map, compactMap etc**: The PromiseKit also included this and some other functional methods methods. This methods are used to change the result of promise. This methods make the code more readble and compact. 

Example:
```swift
firstly {
    URLSession.shared.dataTask(.promise, with: rq)
}.compactMap {
    try JSONSerialization.jsonObject($0.data) as? [String]
}.done { arrayOfStrings in
    //…
}.catch { error in
    // Foundation.JSONError if JSON was badly formed
    // PMKError.compactMap if JSON was of different type
}
```

**firstly**: firstly does not do anything actually. It is a syntactic sugar. It just make the code more readable.

Example:
```swift
firstly {
    login()
}.then { creds in
    //…
}
```

## Xcode Build Configuration Files
