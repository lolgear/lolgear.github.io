—
layout: post
title: "Adventures of F in ObjectiveC: Monoid"
date: 2018-11-25 17:15:34 +0300
categories: objectivec functional monoid
—

# Begin #

This is the very first adventure of Functional Programming ( letter F ) in ObjectiveC world. Long time ago mathematicians invent the Scariest and Dreadful expression of their mind - Haskell language. Most of the people are run away into their OOP world behind walls of patterns and frameworks. But Haskell, the Lord will come one day and reveal you in structure cabin and turn you to the Darkness.

Or you will never understand parallel programming and other immutable stuff, hah.

# Monoid #

Let's start with Monoid definition. 

From [wikipedia](https://en.wikipedia.org/wiki/Monoid).

> Suppose that S is a set and • is some binary operation S × S → S, then S with • is a monoid if it satisfies the following two axioms:
> Associativity
> For all a, b and c in S, the equation (a • b) • c = a • (b • c) holds.
> Identity element
> There exists an element e in S such that for every element a in S, the equations e • a = a • e = a hold.
> In other words, a monoid is a semigroup with an identity element.

You can cry a lot in your oopillow, but it is simple. Well, it is supersimple in terms of interfaces and protocols as you wish. You only need to find a zero element in your group ( i.e. zero in numbers ) and add any operation ( for zero it is addition, because 1 + 0 = 0 + 1 = 1. For one it is multiplication, because 1 * 0 = 0 * 1 = 0 ) for which associativity and identity elements rules are fulfill.

# Protocol #

First of all, we need a protocol for monoid with methods that are necessary for monoid.

```objective-c
@protocol Functional__Monoid
- (id)identityElement;
- (id)operationWithLhs:(id)lhs rhs:(id)rhs;
@end

```
It is not enough for tests, we still can't get any element from monoid. For that we could extend monoid protocol by adding generate method.

@protocol Functional__Monoid__With__Generator <Functional__Monoid>
- (id)generateAny;
@end

Wow, now we can test monoids. But we still don't have type inference. Let's add monoid holder which will add type inference for us. For free. Or not.

```objective-c
@interface SimpleMonoid<T> : NSProxy
@property (strong, nonatomic, readwrite) T <Functional__Monoid__With__Generator> monoid;
+ (instancetype)decoratedInstance:(T <Functional__Monoid__With__Generator>)monoid;
@end

@interface SimpleMonoid<T> (Functional__Monoid__With__Generator) <Functional__Monoid__With__Generator>
- (T)zero;
- (T)operationWithLhs:(T)lhs rhs:(T)rhs;
- (T)generateAny;
@end
```

And implementation.

```objective-c
@implementation SimpleMonoid
- (instancetype)initWithInstance:(id<Functional__Monoid__With__Generator>)monoid {
    self.monoid = monoid;
    return self;
}
+ (instancetype)decoratedInstance:(id<Functional__Monoid__With__Generator>)monoid {
    return [[self alloc] initWithInstance:monoid];
}
@end

@implementation SimpleMonoid (Functional__Monoid__With__Generator)
- (id)identityElement {
    return [self.monoid identityElement];
}
- (id)operationWithLhs:(id)lhs rhs:(id)rhs {
    return [self.monoid operationWithLhs:lhs rhs:rhs];
}
- (id)generateAny {
    return [self.monoid generateAny];
}
@end
```

That's all, we can move forward and find monoids, in, wait for it, simple foundation types.

# Foundation #

I will show you several foundation types which are good candidates for monoids.

They are here

1. NSNumber
2. NSString
3. NSArray
4. NSSet
5. NSDictionary

All of them are monoids, or, kind of.

Let's start with all of them together. In case of our implementation we need a support category  ( no, extension category, not a category in terms of category theory ) in NSObject.

We want type-safe code, right? But parameters doesn't have `instancetype` type. So, we need to reimplement type safety in terms of class conditions.

Our category can be.


```objective-c
@implementation NSObject (TypesafeExtensions)
+ (BOOL)isFineOperandLeft:(id)left right:(id)right {
    return ([left isKindOfClass:self] && [right isKindOfClass:self]);
}
@end

```
Is it enough? Well, we can check both operands for their upper bound type. Yes, it will be enough for our adventure.

Next, you can see all adoptions of protocol `Functional__Monoid__With__Generator`.

Before implementation I would like to add notes about implementation.

Monoid doesn't postulate commutative operation. Hence, you are able to choose any parameter as first. However, operation associativity should take place.

# Implementations #

## NSNumber ##

Numbers are simple and they are immersed or has traits of many mathematics structures. I choose simple addition as an internal operation.

```objective-c
@implementation NSNumber (Functional__Monoid__With__Generator)
- (id)operationWithLhs:(id)lhs rhs:(id)rhs {
    if ([NSNumber checkLeftOperand:lhs andRightOperand:rhs]) {
        __auto_type left = (typeof(self))lhs;
        __auto_type right = (typeof(self))rhs;
        return @(left.doubleValue + right.doubleValue);
    }
    return self.identityElement;
}
- (nonnull id)identityElement {
    return @(0);
}
- (nonnull id)generateAny {
    return [[self.class alloc] initWithInt:arc4random()];
}
@end
```

Our identity element for addition is zero. We simply add right value to left value in operation. And also we generate any element by random function.

## NSString ##

Strings are the first type that doesn't have pretty internal operation like numbers has addition. In case of strings we should choose operation and precedence of it. We can use simple concatenation where right left string is followed by right string. Left + right.

```objective-c
@implementation NSString (Functional__Monoid__With__Generator)

- (nonnull id)operationWithLhs:(nonnull id)lhs rhs:(nonnull id)rhs {
    if ([NSString checkLeftOperand:lhs andRightOperand:rhs]) {
        __auto_type left = (typeof(self))lhs;
        __auto_type right = (typeof(self))rhs;
        return [left stringByAppendingString:right];
    }
    return self.identityElement;
}

- (nonnull id)identityElement {
    return @"";
}

- (nonnull id)generateAny {
    return [NSUUID UUID].UUIDString;
}

@end
```

String is the first class cluster which hides all of subclasses zoo. For that we should check for exactly `NSString` class as superclass of whole class cluster.

## NSArray ##

Arrays are just lists which has concatenation operation. Left + Right. And as `NSString` this class `NSArray` has various subclasses inside. We should check exactly for superclass of this cluster.

```objective-c
@implementation NSArray (Functional__Monoid__With__Generator)

- (nonnull id)operationWithLhs:(nonnull id)lhs rhs:(nonnull id)rhs {
    if ([NSArray checkLeftOperand:lhs andRightOperand:rhs]) {
        __auto_type left = (typeof(self))lhs;
        __auto_type right = (typeof(self))rhs;
        return [left arrayByAddingObjectsFromArray:right];
    }
    return self.identityElement;
}

- (nonnull id)identityElement {
    return @[];
}

- (nonnull id)generateAny {
    return [[[NSString new] generateAny] componentsSeparatedByString:@""];
}

@end
```

Strings and Lists are very similar. If you treat String as List of Characters, so, you automatically get monoid for Strings from monoid for Lists.

## NSSet ##

Sets are pretty awesome as numbers. Their union operation from Set Algebra gives us the same power as addition gives for numbers. However, they are different.

@implementation NSSet (Functional__Monoid__With__Generator)

- (nonnull id)operationWithLhs:(nonnull id)lhs rhs:(nonnull id)rhs {
    if ([NSSet checkLeftOperand:lhs andRightOperand:rhs]) {
        __auto_type left = (typeof(self))lhs;
        __auto_type right = (typeof(self))rhs;
        return [left setByAddingObjectsFromSet:right];
    }
    return self.identityElement;
}

- (nonnull id)identityElement {
    return [self.class new];
}

- (nonnull id)generateAny {
    return [[self.class alloc] initWithArray:[[NSArray new] generateAny]];
}

@end

As you see, we create new set C = A + B in operation. Simple, yes?

## NSDictionary ##

Dictionaries are tricker than others types, because we should prove associativity of operation. But you could enhance current sketch to add associativity constraint to this operation.

```objective-c
@implementation NSDictionary (Functional__Monoid__With__Generator)
- (nonnull id)operationWithLhs:(nonnull id)lhs rhs:(nonnull id)rhs {
    if ([NSDictionary checkLeftOperand:lhs andRightOperand:rhs]) {
        __auto_type left = (typeof(self))lhs;
        __auto_type right = (typeof(self))rhs;
        __auto_type theLeft = [NSMutableDictionary dictionaryWithDictionary:left];
        [theLeft addEntriesFromDictionary:right];
        return theLeft;
    }
    return self.identityElement;
}
- (nonnull id)identityElement {
    return @{};
}
- (nonnull id)generateAny {
    return [NSDictionary dictionaryWithObjects:[[NSArray new] generateAny] forKeys:[[NSArray new] generateAny]];
}
@end
```

We are adding keys and values from one dictionary to another dictionary pair-by-pair. Check that everything will be alright.

# Tests #

Tests are straightforward.

```objective-c
@interface MonoidTests : XCTestCase @end

@implementation MonoidTests

- (NSArray <id<Functional__Monoid__With__Generator>>*)monoids {
    __auto_type number = (SimpleMonoid <NSNumber *>*)[SimpleMonoid decoratedInstance:[NSNumber new]];
    __auto_type string = (SimpleMonoid <NSString *>*)[SimpleMonoid decoratedInstance:[NSString new]];
    __auto_type array = (SimpleMonoid <NSArray *>*)[SimpleMonoid decoratedInstance:[NSArray new]];
    __auto_type set = (SimpleMonoid <NSSet *>*)[SimpleMonoid decoratedInstance:[NSSet new]];
    __auto_type dictionary = (SimpleMonoid <NSDictionary *>*)[SimpleMonoid decoratedInstance:[NSDictionary new]];
    return @[number, string, array, set, dictionary];
}

- (void)testMonoid {
    for (id <Functional__Monoid__With__Generator> monoid in [self monoids]) {
        [XCTContext runActivityNamed:@"Check zero rules" block:^(id<XCTActivity>  _Nonnull activity) {
            __auto_type left = (id)[monoid generateAny];
            __auto_type right = (id)[monoid generateAny];
            __auto_type zero = (id)[monoid identityElement];
            
            // zero rule: ex = xe = x
            // zero rule: ex = xe
            XCTAssertEqualObjects([monoid operationWithLhs:left rhs:zero], [monoid operationWithLhs:zero rhs:left]);
            // zero rule: xe = x
            XCTAssertEqualObjects([monoid operationWithLhs:left rhs:zero], left);
        }];
        
        [XCTContext runActivityNamed:@"Check associativity" block:^(id<XCTActivity>  _Nonnull activity) {
            __auto_type a = (id)[monoid generateAny];
            __auto_type b = (id)[monoid generateAny];
            __auto_type c = (id)[monoid generateAny];
            
            __auto_type a_b = [monoid operationWithLhs:a rhs:b];
            __auto_type b_c = [monoid operationWithLhs:b rhs:c];
            
            // associativity (a + b) + c = a + (b + c)
            XCTAssertEqualObjects([monoid operationWithLhs:a_b rhs:c], [monoid operationWithLhs:a rhs:b_c]);
        }];
    }
}

@end
```

We check identity element rule and associativity of operation.

# Final #

Instead of conclusion I would encourage you to check operation that is defined for NSDictionary against associativity. If something bad happens, you always can change implementation to ensure that new operation has associativity.
