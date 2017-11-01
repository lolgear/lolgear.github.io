---
layout: post
title: "Tricky Swift: Implementation of an accessor with type bounded to current subclass."
date: 2017-11-01 17:15:11 +0300
categories: swift tricks subclasses
---

# Services and Service
In this post I will show you a simple technique that could help you to fight against **Self** return values in terms of subclassing.

Let start from a simple class Services.

```swift
// good start!
class Services {

}
```

But it is too simple. Let's add some services in it.

First of all, we want a services storage and base storage, why not?

```swift
class Service {
    class func name() -> String {
        return ""
    }
    class func typedService() -> Self? {
        return nil
    }
}
```

Let's add `ConcreteService`.

```swift
class ConcreteService: Service {
    override class func name() -> String {
        return "concrete"
    }
}
```

Let try to build something more useful on these assumptions.

# Should we store them together?

And why not? I want to store all services in one place for simple access.

For that reason I add Dictionary which maps service name to stored service.

```swift
class Services {
	var services: [String : Service]!
	init() {
		services = [
			Service.name : Service()
			ConcreteService.name : ConcreteService()
		]
	}
}
```

Ok, we have done it!

But, what if I want to access services by names? Is there any convenient method?

```swift
extension Services {
    func service(by name: String) -> Service? {
    	dict[name]
    }
}
```

But what should I do with it? I will get only `Service?`, not a `ConcreteService`.

Could I somehow get access to concrete types of services.

# Jump back

The first and obvious point to start search is a Service parent class.

Only it could bind `Self?` of its subclasses to correct derived type.

Let write this thought.

```swift
extension Service {
	class func typedService() -> Self? {
		return nil
	}
}
```

Do you remember the intention of `Self` and `instancetype`?

They are preserved keywords which allows to return objects related to concrete type.

That is why you can't cast type to `Self`.

However, could I somehow use generics to make it works right?

Well, let's add class method.

```swift
extension Service {
	class func service<T: Self>() -> T? {
		// does it work?
	}
}
```

Restrictions to `Self` type are allowed only in protocols.

But what if we use concrete type and binding to superclass?

```swift
extension Service {
	class func service<T: Service>() -> T? {
		return self.init() as? T
	}
	class func typedService()-> Self? {
		return service()
	}
}
```
Hey, what've we done now? We just cast `Service` to generic type `T` which has an upper bound ( it should be a subclass of `Service`.

We also cast value of initializer to generic type. What does it mean for our usage? It means that any method which knows its result type, could deduce the result of generic method.

There are three steps to achieve our result.

1. Method with generic result type `T`.
2. `T` bounded to `Service`.
3. Another method with `Self` result type.

# The parameter trick.

This technique could work only in one case - availabilty of casting `T` to `Self`. But how is it possible.

In terms of class and inheritence, the `Self` type defined as concrete subtype of a class, right?

So, `Self` could be restricted to be a superclass, or, in other words, `Self` has an upper bound equal to the upper superclass in class hierarchy.

It means that `Self` has requirement `Self: Service` in our example.

1. `T: Service`
2. `Self: Service`
3. `Service` is a `Service`.

So, in terms of `Service` class, `Self` and `T` as result types are equal and could be propagated to lower subclasses.

# The grand final

W've done the hardest part of our work.

Now, we would like to accumulate knowledge about generics and expand it to `Services`.

```swift
class Services {
	var services: [String : Service]!
	required init() {
		services = [
			Service.name : Service()
			ConcreteService.name : ConcreteService()
		]
	}
}

extension Services {
	    class func defaultServices<T: Services>() -> T? {
        return self.init() as? T
    }
    class func typedDefaultServices() -> Self? {
        return defaultServices()
    }
}

extension Services {
    func service(by name: String) -> Service? {
    	dict[name]
    }
}
```

And some tests.

```swift
extension ConcreteService {
	func candy() {}
}

extension Services {
    class func test() {
        print("AnyService == \(ConcreteService.service())")
        print("ConcreteService == \(ConcreteService.typedService())")
        do {
            Services.typedDefaultServices()?.service(by: ConcreteService.name())?.candy() // don't even compile, because result type is a Service, not a Concrete Service.
        }
        catch let error {
            print("error: \(error)")
        }
    }
}
```

# Wait, it does not work!

At the end I have a services storage and service. They are separated and all easy-to-use type deduction is disappeared.

But what if I could somehow solve it?

Let me think for a moment...

What if `Service` could retrieve himself from `Services` storage?

Something like:

```
extension Service {
	class func service<T: Service>() -> T? {
		return Services.defaultServices()?.service(by: name()) as? T
	}
}
```

Haha! It works!

# Final thoughts

Well, you should know a limits of generics. Most of the time you should not use them without a patience.