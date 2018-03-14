---
layout: post
title: "Tricky Swift: Generic parameter default value."
date: 2018-15-06 01:30:02 +0300
categories: swift tricks generics functions
---

# Begin

Today I am going to answer a simple question for myself.

Is it possible to implement a priority queue template from C++ STL?

More accurately:

> Is there any chance to get the same behavior with default type in generics?

# STL implementation

Let's look at [priority_queue](http://www.cplusplus.com/reference/queue/priority_queue/) from C++ STL.

> template <class T, class `Container` = vector<T>,
> class Compare = less<typename Container::value_type> > class priority_queue;
> It has three template parameters:

1.	Type of element T.
2.	Container type with elements matched Type T.
3.	Compare - binary function which will order elements in priority queue.

From [Wikipedia](https://en.wikipedia.org/wiki/Priority_queue).
> In computer science, a priority queue is an abstract data type which is like a regular queue or stack data structure, but where additionally each element has a "priority" associated with it. In a priority queue, an element with high priority is served before an element with low priority. If two elements have the same priority, they are served according to their order in the queue.

And in other words:
It is a queue which orders elements according to their priorities. If priorities are from [0, B] domain, then, an element with the lowest or the highest priority will be available in O(1) time.

In nutshell, you could easily access max/min element.

Let start to implement something similar.

# Naive implementation

Let us define one protocol that we will use later.

There is no chance that all your elements have `Comparable` ability. For that reason let us define protocol for ordering elements.

```swift
protocol BinaryCompareFunction {
    associatedtype T
    static func compare(lhs: T, rhs: T) -> Bool
}
```

Next obvious step is to define a set of predefined functions that could be used for elements if these elements conform to `Comparable` protocol.

```swift
struct BinaryCompareFunctions {
    struct Less<E>: BinaryCompareFunction where E: Comparable {
        typealias T = E
        static func compare(lhs: E, rhs: E) -> Bool {
            return lhs < rhs
        }
    }
}
```

And one more step to our desired interface. We should define a priority queue which will adopt generic parameters. For one further reason I will wrap it into structure.

```swift
struct PriorityQueues {
    class PriorityQueue<Container: Collection, Comparison: BinaryCompareFunction> where Container.Element == Comparison.T {
        typealias Element = Container.Element
        typealias ComparisonFunction = Comparison
        var container: Container?
        init(container: Container?) {
            self.container = container
        }
    }
    struct test {

    }
}

```
What have we done so far?

```swift
extension PriorityQueues.test {
    static func naive() {
        let _ = PriorityQueues.PriorityQueue<Array<Int>, BinaryCompareFunctions.Less<Int>>(container: [])
    }
}

```
Not so bad. But could we do better?

# Generic default values

## Tips and tricks

Ok, let start to define easy-to-use priority queue with default container `Array`. How could it be done?
You could try to use inheritance or static functions.

1. Class method.
2. Inheritance.
3. Class object.

## Class method

Class method that defined in generic class.

```swift
// MARK: Class method
extension PriorityQueues.PriorityQueue {
    class func defaultQueue<T: Comparable>(t: T) -> PriorityQueues.PriorityQueue<Array<T>, BinaryCompareFunctions.Less<T>> {
        return PriorityQueues.PriorityQueue<Array<T>, BinaryCompareFunctions.Less<T>>(container: [])
    }
}
```

Is it really big and clumsy? We should provide at least one parameter to specialize - type of element. However, function could not infer generic parameter if it appears only in return result. So, for that reason you will add one fake parameter.
What about usage of this monster?

```swift
extension PriorityQueues.test {
    static func classMethod() {
        let i: Int = 0
        let _ = PriorityQueues.PriorityQueue<Array<Int>, BinaryCompareFunctions.Less<Int>>.defaultQueue(t: i)
    }
}
```

Hahaha, it could not be done more neat. You should specialize all parameters, because generic class could not be used without them directly. Also, you should pass fake parameter to a function to eliminate all questions that compiler asks you about generics.

Move one, we could do better!

## Inheritance

We could inherit from our generic class to reduce usage of generics and to add default values for them.

```swift
// MARK: Subclass.
extension PriorityQueues {
    class PriorityQueue_DefaultLess<T: Comparable>: PriorityQueue<Array<T>, BinaryCompareFunctions.Less<T>> {
        convenience init() {
            self.init(container: [])
        }
    }
}
```

Ok, let see what happens. We determine subclass which has default parameter binary compare function with value `BinaryCompareFunction.Less`. Moreover, we also add one necessary restriction for element type. Element should have `Comparable` ability for `BinaryCompareFunction.Less` function.
And usage is pretty simple now.

```swift
extension PriorityQueues.test {
    static func subclass() {
        let _ = PriorityQueues.PriorityQueue_DefaultLess<Int>()
    }
}
```

We only specify our desired element parameter! But what if we do not specify this parameter at all?

```swift
extension PriorityQueues.test {
    static func easy() {
        let _ = PriorityQueues.PriorityQueue_DefaultLess(container: [0])
    }
}
```

Wow! Fantastic! You could say that I forgot about last bullet. No, I do not. Here it is.


## Class object

Well, we could delegate our generic-reduction game to another class and define class method in it.

```swift
// MARK: Class Object
extension PriorityQueues {
    class PriorityQueueObject<T: Comparable> {
        class func defaultQueue() -> PriorityQueue<Array<T>, BinaryCompareFunctions.Less<T>> {
            return PriorityQueue<Array<T>, BinaryCompareFunctions.Less<T>>(container: [])
        }
    }
}
```

Not so bad. We have class that reduces our generic parameter list to one element. We add default value to our binary function and also add `Comparable` restriction to element parameter.
How could we use it? Not a rocket science.

```swift
extension PriorityQueues.test {
    static func classObject() {
        let _ = PriorityQueues.PriorityQueueObject<Int>.defaultQueue()
    }
}
```
