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

Solid is a stable state that keeps form, size and shape of object. As solid object keeps form, size and shape, as SOLID principles keep codebase stable and objective over time.

# Timekeeping devices #

What do these letters mean for us in real life? Which devices have same concepts in real world? Well, I would like to introduce an ordinary device - timekeeping device or watches which easily adapts these SOLID principles.

Let's see each principle under magnifying glass.

# S is Single Responsibility Principle #

> a class should have only a single responsibility (i.e. only changes to one part of the software's specification should be able to affect the specification of the class).

The easiest principle for timekeeping devices. Throughout human history watches just show current time at specific zone.

They've become smarter. Today we have many different types of timekeeping devices, however, they don't boil a water or don't make a dinner. If they do it, it is a hybrid of two devices - one part shows time and another part cooks a dinner. But in the guts they are separated into real different parts. You can open, for example, stove and cut off only a part which shows current time. It is just illusion that they are the same device.

# O is Open and Closed Principle #

> software entities ... should be open for extension, but closed for modification.

Some timekeeping devices have the same mechanism under the hood. For example, one of the oldest devices - a candle.
It is a watch that has a fire mechanism inside. It burns down while time is going on. You can find one hour candle.

Time goes on and this candle has founded a family of different candles and watches with oil mechanism.

Oil watches have interesting addition or extension. They have a transparent vessel which keeps oil and also it has vertical clock face which shows time on different levels. You can have only one oil watch to measure any time between five minutes to one-hour minutes with five minutes interval.

# L is Liskov Substitution Principle #

> objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

It is a simple evolution of watches. Each type of timekeeping device just inherits the main feature of generic timekeeping device - it shows current time.

# I is Interface Segregation Principle #

> many client-specific interfaces are better than one general-purpose interface.
I remember that you read about Single Responsibility principle. Please, read it again.

Timekeeping device is a perfect example of minimalistic device which do its job. Instead of just putting all items like counters, months, days in one device, people create different type of devices in case of diversity of needs. Diversity of needs means interface segregation. You need simple interface that will do its simple work correctly and can be combined with other interfaces.

# D is Dependency Inversion Principle #

> one should "depend upon abstractions, [not] concretions."

It is a tricky principle. However, watches all over the history prove it.

Sand watches, water watches, fire watches, real watches and smartphones. And, for example, you have a check game. 

Obviously, you can use sand watches to play a game. However, it is a bit difficult. You should reset time by turning them upside-down. More interesting, this is not the best option for check game. Or you can use fire clock with "stop" function - just blow it down when you need. It is handful and easy.

But modern devices also developed "ready" function. When you press a button, it will end your turn and start opponent turn.

You use watches as device which shows time and, optionally, has reset function. Design upfront and mechanism underneath are changing all over the time. But you know how to use them because they are functionally clear.

# Final #

Humans are smart enough to developed single-purposed device in real life. Why can't they do it in software development?

Who knows.
