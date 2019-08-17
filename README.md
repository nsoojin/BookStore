# BookStore

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/64a476c06de54ba3be164e14dacf7c30)](https://app.codacy.com/app/nsoojin/BookStore?utm_source=github.com&utm_medium=referral&utm_content=nsoojin/BookStore&utm_campaign=Badge_Grade_Settings)

<p align="left">
    <a href="https://travis-ci.org/nsoojin/BookStore" alt="Build">
        <img src="https://img.shields.io/travis/nsoojin/BookStore/master"/></a>
    <a href='https://coveralls.io/github/nsoojin/BookStore?branch=master'>
        <img src='https://coveralls.io/repos/github/nsoojin/BookStore/badge.svg?branch=master'/></a>
    <a href="https://github.com/nsoojin/BookStore/blob/master/LICENSE" alt="License">
        <img src="https://img.shields.io/github/license/nsoojin/BookStore"/></a>
    <a href="https://github.com/nsoojin/BookStore/pulls">
        <img src="https://img.shields.io/badge/PRs-welcome-brightgreen"></a>
    <a href="https://twitter.com/intent/follow?screen_name=soojinro">
        <img src="https://img.shields.io/twitter/follow/soojinro?style=social"></a>
</p>

See new releases and search for programming books from [IT Bookstore API](https://api.itbook.store)

This is a sample app to practice using `Result` type, stubbing network request for unit tests, separating functionalities into frameworks, and writing Swift documentation.

### How to run

```
> cd BookStore
> open BookStore.xcodeproj
```

Run!

# Contents

- [App Features](https://github.com/nsoojin/BookStore#app-features)

- [`Result` type in Swift 5](https://github.com/nsoojin/BookStore#result-type-in-swift-5)

- [Stubbing Network Requests for Unit Tests](https://github.com/nsoojin/BookStore#stubbing-network-requests-for-unit-tests)

- [Using Frameworks for independent functionalities](https://github.com/nsoojin/BookStore#using-frameworks-for-independent-functionalities)

- [Writing a documentation comment](https://github.com/nsoojin/BookStore#writing-a-documentation-comment)

- [Getting Rid of IUOs](https://github.com/nsoojin/BookStore#getting-rid-of-iuos)

## App Features

### What's New

A simple `UITableView` with cells and modal presentation for a detailed page.

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/whats-new.gif" width="400">

### Search

1. As a user types in the keyword, the search text is "debounced" for a fraction of second for better performance and user experience. See [Debouncer](https://github.com/nsoojin/BookStore/blob/3a3e91f903e5a77ebdcafd53803fd3edad0dde65/BookStoreKit/Utils/Debouncer.swift#L11).

2. Search results are paginated and provides infinite scroll.

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/search.gif" width="400">

## `Result` type in Swift 5

Out of the box, you have to switch on the `Result` instance to access the underlying success instance or the error instance. 

```Swift
switch result {
case .success(let response):
  //do something with the response
case .failure(let error):
  //handle error
}
```

However, I think switch statements are too wordy. I added [`success`](https://github.com/nsoojin/BookStore/blob/5f3d7d85503fd1de51d04b0286653a3578a7125a/BookStore/Extensions/Result%20%2B%20Extensions.swift#L25) and [`catch`](https://github.com/nsoojin/BookStore/blob/5f3d7d85503fd1de51d04b0286653a3578a7125a/BookStore/Extensions/Result%20%2B%20Extensions.swift#L34) method to `Result` type. So it can be chained like this.

```Swift
searchResult.success { response in
  //do something with the response
}.catch { error in
  //handle error
}
```

Even cleaner, like this.

```Swift
result.success(handleSuccess)
      .catch(handleError)
      
func handleSuccess(_ result: SearchResult) { ... }
func handleError(_ error: Error) { ... }
```

## Stubbing Network Requests for Unit Tests

Generally, it is not a good idea to rely on the actual network requests for unit tests because it adds too much dependency on tests. One way to stub networking is to subclass `URLProtocol`.

### 1. Subclass `URLProtocol`

See [`MockURLProtocol`](https://github.com/nsoojin/BookStore/blob/master/BookStoreKitTests/MockAPI/MockURLProtocol.swift)

### 2. Configure `URLSession` with your mock `URLProtocol`

```Swift
let config = URLSessionConfiguration.ephemeral
config.protocolClasses = [MockURLProtocol.self]

//Use this URLSession instance to make requests.
let session = URLSession(configuration: config) 
```

### 4. Use the configured `URLSession` instance just as you would.

```Swift
session.dataTask(with: urlRequest) { (data, response, error) in
  //Stubbed response
}.resume()
```

## Using Frameworks for independent functionalities

Separating your app's functions into targets has several advantages. It forces you to care about dependencies, and it is good for unit tests since features are sandboxed. However, it may slow down the app launch (by little) due to framework loading.

`BookStoreKit` is responsible for fetching and searching books data from [IT Bookstore API](https://api.itbook.store). 

`Networking` is a wrapper around URLSession for making HTTP requests and parsing response.

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/xcodeproj-targets.png" width="200">

## Writing a documentation comment

[Swift's API Design Guidelines](https://swift.org/documentation/api-design-guidelines/#fundamentals) suggest you write a documentation comment for every declaration. Writing one can have an impact on the design.

### 1. Write

Reference this [document](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/index.html) for markup formatting.

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/documentation-0.png">

### 2. Check out the result

In Xcode's autocompletion

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/documentation-1.png" width="400">

and Show Quick Help (option + click)

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/documentation-2.png" width="600">

## Getting Rid of IUOs

IMHO Implictly unwrapped optional is a potential threat to code safety and should be avoided as much as possible if not altogether. An example of two methods to get rid of them from where they are commonly used.

### Make IBOutlets Optional

IBOutlets are IUOs by Apple's default. However, you can change that to Optional types. You may worry that making IBOutlets Optionals may cause too many if lets or guards, but that concern may just be overrated. IBOutlets are mostly used to set values on them, so optional chaining is sufficient. In just few cases where unwrapping is added, I will embrace them for additional safety of my code.

<img src="https://raw.githubusercontent.com/nsoojin/BookStore/master/README_assets/optional-iboutlets.png" width="500">

### Using lazy instantiation

For the properties of UIViewController subclass, IUO can be useful but it's still dangerous. Instead, I use [`unspecified`](https://github.com/nsoojin/BookStore/blob/e27ea7252189e9f7ed2b7a9494334ccab9ce801c/BookStore/Extensions/Unspecified.swift#L11). It generates a crash upon class/struct usage so it can be spotted fast during development, and most importantly no more IUOs.

```swift
//Inside a viewcontroller
lazy var bookStore: BookStoreService = unspecified()
```
