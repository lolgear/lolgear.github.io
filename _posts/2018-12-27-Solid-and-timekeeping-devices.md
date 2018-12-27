---
layout: post
title: "SOLID and timekeeping devices."
date: 2018-12-27 23:04:31 +0300
categories: solid principles metaphores
---

# Begin #

What is a solid? It is a rock that rolls right down to hell. It is a something that can't even go right through the glass. It is something like a stone. It is a solid.

However, physics defines main states of matter: Liquid, solid and gas and plasma. Let's see the definition.

> Solid is one of the four fundamental states of matter (the others being liquid, gas, and plasma). In solids molecules are closely packed. It is characterized by structural rigidity and resistance to changes of shape or volume. Unlike liquid, a solid object does not flow to take on the shape of its container, nor does it expand to fill the entire volume available to it like a gas does. The atoms in a solid are tightly bound to each other, either in a regular geometric lattice (crystalline solids, which include metals and ordinary ice) or irregularly (an amorphous solid such as common window glass). Solids cannot be compressed with little pressure whereas gases can be compressed with little pressure because in gases molecules are loosely packed.

Solid is a stable state that keeps form, size, shape of object. As solid object keeps form, size and shape as SOLID principles keep codebase stable and objective over time.

# SOLID Principles #

> In object-oriented computer programming, SOLID is a mnemonic acronym for five design principles intended to make software designs more understandable, flexible and maintainable.
These letters stands for:
Single responsibility principle 
a class should have only a single responsibility (i.e. only changes to one part of the software's specification should be able to affect the specification of the class).
Open/closed principle
> software entities ... should be open for extension, but closed for modification.
Liskov substitution principle
> objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.
Interface segregation principle
> many client-specific interfaces are better than one general-purpose interface.
Dependency inversion principle
> one should "depend upon abstractions, [not] concretions."

# Timekeeping devices #

What does these letters mean for us in real life? What devices in our life have same concepts? Well, I would like to introduce ordinary device - timekeeping device or watches which easily adapts these solid principles.

Let's see each principle under magnifying glass.

# S is Single Responsibility Principle #

> a class should have only a single responsibility (i.e. only changes to one part of the software's specification should be able to affect the specification of the class).
The easiest principle for timekeeping devices. Throughout human history watches just show current time at specific zone.

Yes, later they became smarter. Today we have many different types of timekeeping devices, however, they doesn't boil a water or make you a dinner or something else. If they do it, it is a hybrid of two devices - one part shows time and another part cooks a dinner. But in the guts they are separated into real different parts. You can open, for example, stove and cut off only a part which shows current time. It is just illusion that they are the same device.

# O is Open and Closed Principle #

> software entities ... should be open for extension, but closed for modification.
And some timekeeping devices do the same in the history. For example, let's find one of the oldest devices - a candle.
It is a watch that has a fire as a mechanism. It burns down while time is going on. You can find one hour candle, for example.

Time goes on and this candle has founded a family of different candles and watches on oil mechanism.

Oil watches has interesting addition or extension. They have a transparent vessel which keeps oil and also it has vertical clock face which shows time on different levels. You can have only one oil watch to measure any time between, for example, five minutes to one-hour minutes with five minutes interval. 

# L is Liskov Substitution Principle #

> objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.
It is simple evolution of watches. Each watch just inherit the main feature of timekeeping device - it shows current time.

# I is Interface Segregation Principle #

> many client-specific interfaces are better than one general-purpose interface.
I remember that you read about Single Responsibility principle. Please, read it again.

Timekeeping device is perfect example of minimalistic device which do its job. Instead of just putting all items like counters, months, days in all devices, people create different type of devices in case of diversity of needs. Diversity in needs means interface segregation. You need simple interface that will do its work correctly and can be combined.

# D is Dependency Inversion Principle #

> one should "depend upon abstractions, [not] concretions."
It is a tricky principle. However, watches all over the history proof it.

For example, you have sand watches, water watches, fire watches, real watches and smartphones. And, for example, you have a check game. 
Obviously, you can use sand watches to play this game. However, it is a bit difficult. You should reset time by turning them upside-down. More interesting, this is not the best mechanism for check game. For example, you can use fire clock with "stop" function - just blow it down when you need. It is handful and easy.
But modern devices also developed "ready" function. When you press your button, it will start opponent turn.
However, you use watches as device which shows time and, optionally, has reset function. Design above and mechanism underneath are changing all over the time. But you know how to use them because they are functionally clear.

# Final #

Humans are smart enough to developed single-purposed device in real life. Why they can't do it in software development?

Who knows.
