---
title: "Locking a view to the top of the keyboard in iOS"
published: true
comments: true
---

Recently I've come across the issue where I need to lock a view to the top of the keyboard when it is displayed in an iOS application. 

This was doable with a small extension to UIView, how ever it didn't offer a lot in the way of customization without adding a decent chunk of code.

So, I set to work on [KeyboardLock](https://github.com/nathan-fiscaletti/KeyboardLockiOS), a simple module for iOS apps that will do most of the work for you.

![](https://github.com/nathan-fiscaletti/KeyboardLockiOS/raw/master/Images/preview.gif)

## Installation

KeyboardLock is available through [CocoaPods](https://cocoapods.org/). To install it, simply add the following line to your Podfile:
    
```shell
$ pod 'KeyboardLock'
```

## Usage

Implementing a keyboard lock is as simple as the following
    
```swift
KeyboardLock(
    withView: containerView,
    andLockType: .BottomConstraint
).lock()
```

[See the GitHub Repository](https://github.com/nathan-fiscaletti/KeyboardLockiOS)