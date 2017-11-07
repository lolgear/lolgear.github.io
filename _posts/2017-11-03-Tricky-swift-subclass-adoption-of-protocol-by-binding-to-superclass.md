---
layout: post
title: "Tricky Swift: Subclass adoption of protocol by binding to superclass."
date: 2017-11-03 16:56:48 +0300
categories: swift tricks subclasses
---

# How to debug different things?

Well, I have several questions about protocol `CustomDebugStringConvertible`. Is it really enough to describe all variates and types of information? Is it really cool if all debug data is a string?

You could answer on this question as you wish, I just show an illustration of another protocol which helps you to present debug data differently.

# Introduction

Firts of all, assume that you have a complex object which you want to debug. For example, you start debug `Core Data Entity` or an `UIView`. And when you stop process execution at breakpoint, you take a deep breath and look at debug inspector. It is not a best place to dig into. You would like to group all related data in one method and print to console it.  
For example, I would like to know the count of subviews or the types or any information about its sizes. And it is only the information that I would like to know.  
Or I would like to be able to inspect only a group of fields in `Core Data Entity` by input only one method.

Is it a good decision to use `DebugDescription`?

# A Debug Protocol

Let me add another protocol

```swift
protocol DebugInformationProtocol {
    typealias InformationType = [AnyHashable: CustomDebugStringConvertible]
    func debugInformation() -> InformationType
}
```

This protocol could show you a `debugInformation` about an object. Moreover, It could show you a set of different information about an object. You could add what you want.

Without loss of generality we could adopt this item by `NSObject` as an ancestor of `UIView` and `NSManagedObject`.

```swift
extension NSObject: DebugInformationProtocol {
    func debugInformation() -> DebugInformationProtocol.InformationType {
        return ["debugDescription": debugDescription]
    }
}
```

That's all! You has one method for all cases of debugging process? What if you want to extend this protocol by only one method for debug only for a specific subclass or subclass hierarchy. `NSObject` hierarchy is so huge!

# Extension only for a subset of subclasses

Suppose, that you would like to add different methods for `UIView` hierarchy and for `NSManagedObject` hierarchy.

For example, you would like to output a count of UIButtons in its subviews layer. Or you would like to output `attributesByName` property for `NSManagedObject` subclasses.

Let's do it via protocol extension!

```swift
extension DebugInformationProtocol where Self: UIView {
    func debugInformationAboutSubviews() -> Int {
        return self.subviews.filter { $0 is UIButton }.count
    }
}

extension DebugInformationProtocol where Self: NSManagedObject {
    func debugInformationAboutEntity() -> InformationType {
        return entity.attributesByName
    }
}
```

Don't forget about some tests.

```swift
class CustomControl: UIView {
    func test() {
        self.debugInformationAboutSubviews()
        self.debugInformation()
    }
}

class DatabaseObject: NSManagedObject {
    func test() {
        self.debugInformationAboutEntity()
        self.debugInformation()
    }
}
```

# The Conclusion

An interesting protocol feature was described in this post. You could expand protocols on any class by adding constraint which binds Self to custom class hierarchy.

This is a test case which shows a desired behavior.

```swift
class DebugInformationTests {
    class CustomControl: UIView {
        func test() {
            self.debugInformationAboutSubviews()
            self.debugInformation()
        }
    }
    
    class DatabaseObject: NSManagedObject {
        func test() {
            self.debugInformationAboutEntity()
            self.debugInformation()
        }
    }
}
```