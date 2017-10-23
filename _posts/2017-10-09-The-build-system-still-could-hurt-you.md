---
layout: post
title: "The build system could hurts you."
date: 2017-09-18 16:16:01 -0600
categories: build-configurations frameworks
---


# Long time ago

Apple released new developer tool - Swift language. It intends to be a replacement of pretty old and clumsy Objective-C language.

# Frameworks and Static libraries

After swift released, it obligues you to develop libraries with one additional requirement - libraries should be frameworks.

Tons of information are available on Apple developer, Stackoverflow. and third-party sites.  

Maybe you could get information by listening your CS professor.

Nevermind, I just tell you one simple thing.

Dynamic libraries are hard, very hard to embed in each other in iOS terms (it is forbidden).  
Thus, you should not to embed them. 

Never.

# Long way to Union

Let me start from an example application. Whatever application that offer some functionality.

For example, you are developing brand new binary protocol for brand new chat.

Let me assume that you divide application into modules. I hope so that you did it before reading these lines.

However, if you don't do it, let me do it for you.

I will extract DatabaseModule, NetworkModule and ProtocolModule. Maybe I forget something but for me it would be enough for now.

# Last minute before journey begins

Ok, let start discuss how many bugs I gathered in a long journey to simple conquer-and-divide principle.

Apple announced Swift and frameworks. Embedded or not, they are frameworks.

I would like to share with you some troubles and solutions that I have seen during my investigation of all variations of embedded frameworks.

# The crossroad

The main decision that we do in application development is how we are going to keep dependencies.

| The variant | Troubles |
| --- | --- |
| git submodule | Up-to-date complex setup |
| dependencies manager | Limits of dependencies manager |
| third-party binary library | No updates |

## Git Submodule

The most underrated solution due to its complexity. It is untrue. However, modern way is to use dependencies by managers not source code itself. ["How do I remove a submodule?"](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule) is one of the famous questions on Stackoverflow. If you wish - use it, it has advantages and disadvantages.

However, it is pure example in terms of dependencies. You don't need another stuff to compile project.

## Third-party binary library

The binary library is what you are looking for if you build software on top of a platform.

If you are `iOS` or `.NET` or `Android` developer, most of the time you don't need to access product version. It doesn't mean that you are free from this knowledge. You should know about certain limitations of features or bugs that exist on specific platform version.

But you don't need to know source code of these items, it is not your area of interest in current product or project or position. Even if you are core developer and contribute to Swift or JVM, it is not a standard practice to build JVM or Swift for certain product and use it as source code instead of binary.

However, you could do, remember, no limitations in sofware development world.

Before I treated library only as binary and choose a core or platform or system library for example.

"Hey, you tell about third-party libraries?!" - you could say.

Yes, I tell about third-party libraries cause binary libraries of platform are already shipped onto specific platform.

The difference between third-party libraries and system libraries is a delivery. They should be delivered to a project and be consistent with its latest stable version in certain compatibility conditions.

Let me finish telling you obvious stuff about libraries and compatilibity.
 
Third-party vendor gives you a binary and you should manage it, it is a point.

## Dependencies manager.

Dependencies manager. It feels like a superpower.

"I setup dependencies manager tonight, cool?" - you could say to a person from different IT area.  
"Cool!" - was a response.

In iOS development world there are two of them that I know - [Carthage](https://github.com/Carthage/Carthage) and [CocoaPods](https://github.com/CocoaPods/CocoaPods). I don't forget about [Swift Package Manager](https://swift.org/package-manager/), it is still in development and hard to use.

Carthage is a decentralized dependencies manager. CocoaPods is a centralized dependencies manager with a specs repository to store all available dependencies.

### Carthage
Decentralized dependencies manager which supports only frameworks. It builds artefacts in its directory and allow you to link them by hands. It has Carthage dependencies file ( `Cartfile` ) which store all information about how to find and build your libraries.

You should use links to existing github repository in Cartfile. As I remember, you could specify tag, commit and branch with it. 

Good solution, yes?

You could change mind after several tests that I will do.

### CocoaPods
Centralized dependencies manager which supports both frameworks and static libraries. You switch between them by adding `use_frameworks!` directive. It has Pods dependencies file ( `Podfile` ) and additionaly spec dependency file ( `.podspec` ).

The whole process is very simple. It has many challenges that are hidden under nice command-line facade.

Instead of `Carthage`, which build dependencies **only once**, `CocoaPods` doesn't build them at all, allow you to build them on build action. I want to name it `sources dependencies manager` for this specific feature. But I will not use it later.

To post a library to CocoaPods spec repository, you need a `.podspec`. It has meta information about a contributors and map options that allow pod engine to gather source code from a repository and connect it to your project as source code, not a binary.

To get necessary dependencies, you need to configure `Podfile` file which containts information about your project structure. In case of automation, you need to specify existing targets and their names, workspace files and project files. Yes, if it could, it will substitute default names.

Dependencies, contrary to `Carthage`, could be installed by name with optional additional information as tag, branch or commit.

### Comparison table

Dependencies Manager comparison table.

| Feature | CocoaPods | Carthage |
| --- | --- | --- |
| Centralized | Centralized | Decentralized |
| Build artefacts once | No | Yes |
| Build artefacts before build | No | Yes |
| Supports iOS prior version 8 | Yes | No |
| Supports static libraries | Yes | No |
| Linking | Automatic | Manual |
| Dependencies file | Podfile | Cartfile |
| Dependency file | `.podspec` | None |
