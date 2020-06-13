---
layout: post
title: "Tricky Swift: Namespaces as fileprivate typealiases"
date: 2020-06-09 13:43:00 +0300
categories: swift tricks namespaces access-control
---

# Introduction #

Since Swift doesn't have namespaces on project level and requires that every component should have its own framework or target, you could do the following trick.

Define file private type alias to your namespace.

# Begin #

Consider a scenario where you want to move models from old models to new models. So, you want to upgrade implementation.

You can use enums as namespaces trick to separate old models from new models. You may have something like this:

```swift
/// Models.swift
enum Models {
    enum Old {}
    enum New {}
}
```

```swift
/// Models+Old.swift
extension Models.Old {
    class Address {}
    class City {}
}

extension Models.Old.Address: AddressProtocol {}
extension Models.Old.City: CityProtocol {}
```

```swift
/// Models+New.swift
extension Models.New {
    class Address {}
    class City {}
}

extension Models.New.Address: AddressProtocol {}
extension Models.New.City: CityProtocol {}
```

As you see, you should copy-paste a lot of data. In my example I intentionally use classes instead of structures. If you have a structure, you already have namespaces of such kind. With classes you may want to subclasses from old class and redefine methods or you want to extend classes and add several methods.

But what if you want to change classes to structures or vice versa? It is not possible to inherit from structure or to be inherited by structure. It makes no sense.

What can we do?

# Namespaces as typealiases #

Our approach is simple. We just define a type alias to our namespace in each file. That's it. But also change access control for this type alias to file private.

Consider previous scenario with our type aliases.

```swift
/// Models.swift
enum Models {
    enum Old {}
    enum New {}
}
```

```swift
/// Models+Old.swift
fileprivate typealias Namespace = Models.Old
extension Namespace {
    class Address {}
    class City {}
}

extension Namespace.Address: AddressProtocol {}
extension Namespace.City: CityProtocol {}
```

```swift
/// Models+New.swift
fileprivate typealias Namespace = Models.New
extension Namespace {
    class Address {}
    class City {}
}

extension Namespace.Address: AddressProtocol {}
extension Namespace.City: CityProtocol {}
```

That's it!

# Warnings #

These virtual namespaces don't replace types. You still need to address to original type with correct access control level if you want to use types in either internal or public or open or any weaker access control level.

If you define, for example, conformance to Equatable protocol, you should use original types.

Consider content of the following file:

```swift
/// Models+Old+Equatable.swift
fileprivate typealias Namespace = Models.Old

extension Namespace.Address: Equatable {
  static func == (lhs: Models.Old.Address, rhs: Models.Old.Address) -> Bool { false }
}
```

You could try to change types of equality operator, but it doesn't allow you to do so. Because method should be declared as private as its signature types. And our types would have namespace access control level. So they would be file private.

# Conclusion #

Described technique allows you to write quick migrations of your models. You can copy and paste files and just change methods instead of types. You change types, actually, in one place. In namespace.
