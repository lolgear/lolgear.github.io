---
layout: post
title: "Mutability and copying"
date: 2019-01-27 17:51:45 +0300
categories: objc mutability properties copying
---

# Introduction #

Several days ago I was asked about implementation of one of simplest interfaces - simple structure in Objective-C. I a bit confused about interface because it doesn't have proper storage modifiers on properties. Moreover, I was treated as "person who don't know about mutable classes."

Let's dive in.

# Mutability #

Objective-C uses mutable counterparts of various classes to narrow down area of their mutable states and also to improve performance of various operations. However, mutable classes have their trade-offs as improper storage.

Since many mutable classes are subclases or sons of their immutable superclasses and fathers, it is necessary to keep them separately and assure yourself that this object in this variable is mutable or immutable, no matter what is happening behind scenes.

# Jumps #

Yeah, strange word, I should entitle this part as bridges/conversion/interoperability (kind of)/transforms etc. But, wait, it is really boring.

We have two classes: immutable superclass and mutable subclass. Let's give them their names.

```objective-c
@interface ImmutableSuperclass : NSObject
@end
@Interface MutableSubclass : ImmutableSuperclass
@end
```

We can also define somekind of protocol for jumps between them. But it is rather complex for our purposes.

So said Apple in their NSObject and gives us a beautiful protocol NSCopying which provides necessary functionality. Nor far so good for us, it only gives copy methods. For that Apple separate mutable copy methods into another protocol NSMutableCopy.


# Protocols #

We have mutable classes in Foundation that adopts NSCopying and NSMutableCopying protocols. But since mutable classes are subclasses of immutable classes, immutable classes must adopt both protocol.

Conversion scheme is easy. You ask immutable object to provide a mutable object by mutable copying. And vice versa. Mutable object can provide an immutable object by immutable or general copying.

I emphasize that general or immutable copying is not the same as mutable one. Most of the time you would like to have structures that can give copies without any features of transforms and mutability.

Mutable copying is closer to mutabilty pattern. It should give you an object which can mutate. No matter what you have, mutable copy will give you open to modifications object.

But, rules are rules, you can break them.



# Example #

Let's discuss simple example with ordinary structure in Objective-C.

```objective-c
@interface Portfolio : NSObject
@property (nonatomic, readwrite) NSString *name;
@property (nonatomic, readwrite) NSString *ID;
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name;
@end
```

Something wrong here? No, wait for the implementation.

```objective-c
@implementation Portfolio
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (self = [super init]) {
        self.ID = ID;
        self.name = name;
    }
    return self;
}
@end
```

Now you will throw rotten tomatoes at me. Strings can be mutable and we can update them, of course, where we can.
But hold for a second. Is there something really bad and what exactly I declare here. Ok, I will give you my chick but with one condition - do not change interface.

# Tomatoes #

Tomatoes are in hands and are ready for my chick. Let me see what you can.

```objective-c
@interface Portfolio : NSObject
@property (nonatomic, readwrite) NSString *name;
@property (nonatomic, readwrite) NSString *ID;
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name;
@end
```

Remember? I ask you not to change implementation. And tomatoes are thrown.

```objective-c
@implementation Portfolio
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (self = [super init]) {
        _ID = [ID copy];
        _name = [name copy];
    }
    return self;
}
@end
```

Nice try, my chick is on fire. However, let me give your tomato back.

# Backdraft #

Yeah, example is fixed, you are great and all. My suggestion would be simple. Consider an ID subclass of NSString.

```objective-c
@interface Portfolio_Identifier : NSString
- (instancetype)initWithUUID:(NSUUID *)uuid;
@end

@interface Portfolio_Identifier ()
@property (copy, nonatomic, readwrite) NSString *UUID;
@end

@implementation Portfolio_Identifier
- (instancetype)init {
    return [self initWithUUID:[NSUUID UUID]];
}
- (instancetype)initWithUUID:(NSUUID *)uuid {
    if (self = [super init]) {
        self.UUID = [uuid.UUIDString stringByReplacingOccurrencesOfString:@"-" withString:@""];
    }
    return self;
}
- (instancetype)copyWithZone:(NSZone *)zone {
    return [[self.class alloc] init];
}
- (NSUInteger)length {
    return self.UUID.length;
}
- (NSString *)debugDescription {
    return self.UUID;
}
- (NSString *)description {
    __auto_type length = self.length;
    __auto_type notHiddenUntilIndex = 3;
    __auto_type notHiddenAfterIndex = 3;
    __auto_type range = NSMakeRange(notHiddenUntilIndex, length - notHiddenUntilIndex - notHiddenAfterIndex);
    return [self.UUID stringByReplacingCharactersInRange:range withString:@"***"];
}
@end
```

And run tests.

    __auto_type uuid = [NSUUID UUID];
    __auto_type ID = [[Portfolio_Identifier alloc] initWithUUID:uuid];
    __auto_type portfolio = [[Portfolio alloc] initWithID:ID name:@"Name"];
    NSLog(@"developer will see: %@", portfolio.debugDescription);
    NSLog(@"other will see: %@", portfolio);
    XCTAssertNotNil(portfolio);

Interesting feature here I have added. You can not copy Identifier string here, because it will generate another UUID. And this is correct behavior.

# Next Steps #

As far as I know, copying string without any necessity is a bad demeanor. My interface intentionally said that name and identifier have strong storage modifier by not specifying any other.
You have these modifiers to provide information not only to compiler, but also for human.



# Copying everything #

Consider following extension. You should also check that ID and name are not nil. And also, please, adopt copying protocol to portfolio.

```objective-c
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (id == nil || name == nil) { return nil; }
    if (self = [super init]) {
        _ID = [ID copy];
        _name = [name copy];
    }
    return self;
}

- (void)setName:(NSString *)name {
	if (name == nil) { return; }
	_name = [name copy];
}

- (void)setID:(NSString *)ID {
	if (ID == nil) { return; }
	_ID = [ID copy];
}

- (id)copyWithZone:(NSZone *)zone {
    return [[self.class alloc] initWithID:self.ID name:self.name];
}
```

But we can't copy ID. Let's fix this implementation by removing all copy occurrences.

```objective-c
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (id == nil || name == nil) { return nil; }
    if (self = [super init]) {
        _ID = ID;
        _name = [name copy];
    }
    return self;
}

- (void)setID:(NSString *)ID {
	if (ID == nil) { return; }
	_ID = ID;
}
```

It is still incorrect. Because I can pass mutable string or general string as ID. Should we also check for them? Also, do we have access to our new subclass in portfolio? Let's try to figure out what we can do?

```objective-c
- (NSString *)copyIfNeeded:(NSStirng *)string {
	BOOL shouldCopy = string == nil || [string isKindOf:NSMutableString.class];
	return shouldCopy ? [string copy] : string;
}

- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (id == nil || name == nil) { return nil; }
    if (self = [super init]) {
        _ID = [self copyIfNeeded:ID];
        _name = [name copy];
    }
    return self;
}

- (void)setID:(NSString *)ID {
	if (ID == nil) { return; }
	_ID = [self copyIfNeeded:ID];
}

```
Much better. Let me see what should I do in my variant.

```objective-c
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (id == nil || name == nil) { return nil; }
    if (self = [super init]) {
        self.ID = ID;
        self.name = name;
    }
    return self;
}

- (void)setID:(NSString *)ID {
	if (ID == nil) { return; }
	_ID = [self copyIfNeeded:ID];
}

```
Hm, I think that my variant is much more simple than yours.

# Relax #

What if we can change property modifiers of this interface?

```objective-c
@interface Portfolio : NSObject <NSCopying>
@property (copy, nonatomic, readwrite) NSString *name;
@property (copy, nonatomic, readwrite) NSString *ID;
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name;
@end

```
Oh no, now we can't use our custom NSString subclass. It will have invalid behavior. But in property modifiers this is an actual state of consistency. So, we need to copy strings.

Your variant will be a bit easier.

```objective-c
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (id == nil || name == nil) { return nil; }
    if (self = [super init]) {
        _ID = [ID copy];
        _name = [name copy];
    }
    return self;
}

- (void)setID:(NSString *)ID {
	if (ID == nil) { return; }
	_ID = ID;
}

```
My variant will be a lot easier. ( Joking ). 

```objective-c
- (instancetype)initWithID:(NSString *)ID name:(NSString *)name {
    if (id == nil || name == nil) { return nil; }
    if (self = [super init]) {
        self.ID = ID;
        self.name = name;
    }
    return self;
}

- (void)setID:(NSString *)ID {
	if (ID == nil) { return; }
	_ID = ID;
}
```


# Conclusion #

Properties modifiers can be a good or an evil, but they provide information not only to compiler, but also for human. Do not blindly use them without full understanding of their powers. In this article you have found that even simple copying can play some tricks with you.

And when another senior developer tells you about wrong interfaces or implementations, point his nose to this article and ask him.

- Are you still convinced in copy everything?
