## VIPER Architecture

VIPER architecture consists of five layers of components. Each of the letters of VIPER represents one of the layers. Lets take a look at the layers:

1. **View (Views + ViewControllers):** This layer is comprised of Views and ViewControllers. All the view presentation logics are inside this layer.
2. **Interactor:** Business logics for each specific use cases are in the Interactor layer.
3. **Presenter:** Presenter layes between the Interactor and View layer. It prepares Interactor Data to be used by View. It is sort of ViewModel in MVVM.
4. **Entity (Model):** This layers comprises of the Stucts and Classes to represent the data used by the App.
5. **Router (Coordinator):** This layer manages all the navigation logic of the App.


thanks:
* https://github.com/nodes-ios/Playbook/blob/master/ViperArchitecture.md

## PromiseKit

**PromiseKit** is a library that helps with asynchronous programming in swift and Objective-C. It is a wrapper around async task. Instead of using completion and error handler, a method can return a Promise.
It is explained as "PromiseKit is a thoughtful and complete implementation of promises" in the official doc.

Advantage of using PromiseKit over traditional iOS async programming is:
* Cleaner and readable code
* Quick learning carve 
* Very easy to call multiple async functions and managing it
* Save us from Callbacks Pyramid of Doom. Nested callbacks are now in chain.
* Error Handling is separated. Duplicate multiple error handlings are in one place.

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

**Creating a promise**:
```swift
func getWeather(
  atLatitude latitude: Double, 
  longitude: Double
) -> Promise<WeatherInfo> {
  return Promise { seal in
    let urlString = "http://api.openweathermap.org/data/2.5/weather?" +
      "lat=\(latitude)&lon=\(longitude)&appid=\(appID)"
    let url = URL(string: urlString)!

    URLSession.shared.dataTask(with: url) { data, _, error in
      guard let data = data,
            let result = try? JSONDecoder().decode(WeatherInfo.self, from: data) else {
        let genericError = NSError(
          domain: "PromiseKitTutorial",
          code: 0,
          userInfo: [NSLocalizedDescriptionKey: "Unknown error"])
        seal.reject(error ?? genericError)
        return
      }

      seal.fulfill(result)
    }.resume()
  }
}
```

**Threading**: By default all PromiseKit handlers run on main queue. All handler take a on parameter where you can specify the queue.

Example:
```swift
class MyRestAPI {
    func avatar() -> Promise<UIImage> {
        let bgq = DispatchQueue.global(qos: .userInitiated)

        return firstly {
            user()
        }.then(on: bgq) { user in
            URLSession.shared.dataTask(.promise, with: user.imageUrl)
        }.compactMap(on: bgq) {
            UIImage(data: $0)
        }
    }
}
```

**Do I need to use [weak self] in PromiseKit block?**
"Generally, no. Once a promise completes, all handlers are released and so any references to self are also released." Details: https://github.com/mxcl/PromiseKit/blob/master/Documentation/FAQ.md#do-i-need-to-worry-about-retain-cycles


Thanks and additional resources:
* https://github.com/mxcl/PromiseKit/tree/master/Documentation
* https://agostini.tech/2018/10/08/using-promisekit/
* https://www.raywenderlich.com/9208-getting-started-with-promisekit

## Xcode Build Configuration Files

## UISearchController
