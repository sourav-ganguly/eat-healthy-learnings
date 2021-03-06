# Eat_Healthy_Learnings

## VIPER Architecture

## PromiseKit

**PromiseKit** is a library that helps with asynchronous programming in swift and Objective-C. It is explained as "PromiseKit is a thoughtful and complete implementation of promises" in the official doc.

Advantage of using PromiseKit over traditional iOS async programming is:
* Cleaner and readable code
* Easy learning carve 
* Easier and cleaner syntax when work with multiple asynchronous call

Example: 

```
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

## Xcode Build Configuration Files
