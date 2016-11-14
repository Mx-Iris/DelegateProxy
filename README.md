![DelegateProxy](Assets/DelegateProxy_Logo.png)  

<p align="center">
<a href="https://travis-ci.org/ra1028/DelegateProxy"><img alt="Build Status" src="https://travis-ci.org/ra1028/DelegateProxy.svg?branch=master"/></a>
<a href="https://developer.apple.com/swift"><img alt="Swift3" src="https://img.shields.io/badge/language-swift3-orange.svg?style=flat"/></a>
<a href="http://cocoadocs.org/docsets/DelegateProxy"><img alt="Platform" src="https://img.shields.io/cocoapods/p/DelegateProxy.svg?style=flat"/></a><br>
<a href="https://cocoapods.org/pods/DelegateProxy"><img alt="CocoaPods" src="https://img.shields.io/cocoapods/v/DelegateProxy.svg"/></a>
<a href="https://github.com/Carthage/Carthage"><img alt="Carthage" src="https://img.shields.io/badge/Carthage-compatible-yellow.svg?style=flat"/></a>
<a href="https://codebeat.co/projects/github-com-ra1028-delegateproxy"><img alt="Codebeat" src="https://codebeat.co/badges/2cf2f4f3-2c4a-4999-aeb3-ca392d559dc6" /></a>
<a href="https://github.com/ra1028/DelegateProxy/blob/master/LICENSE.md"><img alt="Lincense" src="http://img.shields.io/badge/license-MIT-000000.svg?style=flat"/></a>
</p>  

<H4 align="center">Proxy for receive delegate events more practically</H4>  

---

## About DelegateProxy  
__DelegateProxy__ enable you to receive delegate events by subscribed handler.  

This is generic version of DelegateProxy by [RxSwift](https://github.com/ReactiveX/RxSwift)  

It means be able to use in combination with any other reactive-framework like [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) or [SwiftBond](https://github.com/SwiftBond/Bond), etc.  

---

## Requirements
- Swift 3 / Xcode 8
- OS X 10.9 or later
- iOS 8.0 or later
- watchOS 2.0 or later
- tvOS 9.0 or later

---

## Installation

### [CocoaPods](https://cocoapods.org/)  
Add the following to your Podfile:
```ruby
use_frameworks!

target 'YOUR_TARGET_NAME' do
  pod 'DelegateProxy'
end
```
```sh
$ pod install
```

### [Carthage](https://github.com/Carthage/Carthage)  
Add the following to your Cartfile:
```ruby
github "ra1028/DelegateProxy"
```
```sh
$ carthage update
```

---

## Basic Example
Create `DelegateProxy` inherited class.  
```Swift
final class ScrollViewDelegateProxy: DelegateProxy, UIScrollViewDelegate, DelegateProxyType {
    func resetDelegateProxy(owner: UIScrollView) {
        owner.delegate = self
    }
}
```
It can be useful to implements extension.  
```Swift
extension UIScrollView {
    var delegateProxy: DelegateProxy {
        return ScrollViewDelegateProxy.proxy(for: self)
    }
}
```
You can receive delegate events as following.  
```Swift
scrollView.delegateProxy
    .receive(#selector(UIScrollViewDelegate.scrollViewDidScroll(_:))) { args in
        guard let scrollView: UIScrollView = args.value(at: 0) else { return }
        print(scrollView.contentOffset)
}
```

---

## Custom Example
You can receive delegate events by `Receivable` protocol implemented class.  
Followings are examples of use DelegateProxy with reactive-frameworks.  

#### With [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
Create receiver class.  
```Swift
final class RACReceiver: Receivable {
    let (signal, observer) = Signal<Arguments, NoError>.pipe()

    func send(arguments: Arguments) {
        observer.send(value: arguments)
    }
}
```
Extension  
```Swift
extension DelegateProxy {
    func rac_receive(selector: Selector) -> Signal<Arguments, NoError> {
        return RACReceiver().subscribe(to: self, selector: selector).signal
    }
}
```
Receive events by streams.  
```Swift
scrollView.delegateProxy
    .rac_receive(selector: #selector(UIScrollViewDelegate.scrollViewDidScroll(_:)))
    .map { $0.value(at: 0, as: UIScrollView.self)?.contentOffset }
    .skipNil()
    .observeValues { print("ContentOffset: \($0)") }
```

#### With [SwiftBond](https://github.com/SwiftBond/Bond)
Create receiver class.  
```Swift
final class BondReceiver: Receivable {
    let subject = PublishSubject<Arguments, NoError>()

    func send(arguments: Arguments) {
        subject.next(arguments)
    }
}
```
Extension  
```Swift
extension DelegateProxy {
    func bnd_receive(selector: Selector) -> Signal<Arguments, NoError> {
        return BondReceiver().subscribe(to: self, selector: selector).subject.toSignal()
    }
}
```
Receive events by streams.  
```Swift
scrollView.delegateProxy
    .bnd_receive(selector: #selector(UIScrollViewDelegate.scrollViewDidScroll(_:)))
    .map { $0.value(at: 0, as: UIScrollView.self)?.contentOffset }
    .ignoreNil()
    .observeNext { print("ContentOffset: \($0)") }
```

---

## Contribution  
Welcome to fork and submit pull requests!!  

Before submitting pull request, please ensure you have passed the included tests.  
If your pull request including new function, please write test cases for it.  

---

## License  
DelegateProxy is released under the MIT License.  
