---
layout: post
title: "The build system could hurt you."
date: 2017-10-23 18:56:46 +0300
categories: build-configurations frameworks
---


# Long time ago

Apple released new developer tool - Swift language. It intends to be a replacement of pretty old and clumsy Objective-C language.

# Frameworks and Static libraries

After swift released, it obliges you to develop libraries with one additional requirement - libraries should be frameworks.

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

# The Crossroad

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

However, you could do, remember, no limitations in software development world.

Before I treated library only as binary and choose a core or platform or system library for example.

"Hey, you tell about third-party libraries?!" - you could say.

Yes, I tell about third-party libraries cause binary libraries of platform are already shipped onto specific platform.

The difference between third-party libraries and system libraries is a delivery. They should be delivered to a project and be consistent with its latest stable version in certain compatibility conditions.

Let me finish telling you obvious stuff about libraries and compatibility.
 
Third-party vendor gives you a binary and you should manage it, it is a point.

## Dependencies manager.

Dependencies manager. It feels like a superpower.

"I setup dependencies manager tonight, cool?" - you could say to a person from different IT area.  
"Cool!" - was a response.

In iOS development world there are two of them that I know - [Carthage](https://github.com/Carthage/Carthage) and [CocoaPods](https://github.com/CocoaPods/CocoaPods). I don't forget about [Swift Package Manager](https://swift.org/package-manager/), it is still in development and hard to use.

Carthage is a decentralized dependencies manager. CocoaPods is a centralized dependencies manager with a specs repository to store all available dependencies.

### Carthage
Decentralized dependencies manager which supports only frameworks. It builds artifacts in its directory and allow you to link them by hands. It has Carthage dependencies file ( `Cartfile` ) which store all information about how to find and build your libraries.

You should use links to existing github repository in `Cartfile`. As I remember, you could specify tag, commit and branch with it. 

Good solution, yes?

You could change mind after several tests that I will do.

### CocoaPods
Centralized dependencies manager which supports both frameworks and static libraries. You switch between them by adding `use_frameworks!` directive. It has Pods dependencies file ( `Podfile` ) and additional spec dependency file ( `.podspec` ).

The whole process is very simple. It has many challenges that are hidden under nice command-line facade.

Instead of `Carthage`, which build dependencies **only once**, `CocoaPods` doesn't build them at all, allow you to build them on build action. I want to name it `sources dependencies manager` for this specific feature. But I will not use it later.

To post a library to CocoaPods spec repository, you need a `.podspec`. It has meta information about a contributors and map options that allow pod engine to gather source code from a repository and connect it to your project as source code, not a binary.

To get necessary dependencies, you need to configure `Podfile` file which contains information about your project structure. In case of automation, you need to specify existing targets and their names, workspace files and project files. Yes, if it could, it will substitute default names.

Dependencies, contrary to `Carthage`, could be installed by name with optional additional information as tag, branch or commit.

### Comparison table

Dependencies Manager comparison table.

| Feature | CocoaPods | Carthage |
| --- | --- | --- |
| Centralized | Centralized | Decentralized |
| Build artifacts once | No | Yes |
| Build artifacts before build | No | Yes |
| Supports iOS prior version 8 | Yes | No |
| Supports static libraries | Yes | No |
| Linking | Automatic | Manual |
| Dependencies file | Podfile | Cartfile |
| Dependency file | `.podspec` | None |

## What have I done!

I mention above that my application has several modules. It is ObjectiveC application with zero Swift. ( Cola zero, yes ). It uses `CocoaPods` dependencies manager and it has zero modules or frameworks targets. Nice beginning.

I start to split application into different modules. ( I don't use past time because it could be done now by you ). Several issues will occur, of course.

### Description tables

Let me show all issues that I catch in my divide-and-conquer march.

This table show all states of my project configuration in timeline.

| # | Project configuration name | Description |
| ---- | ---- | ---- |
| 1 | Static libraries and one project target | It has zero frameworks or framework targets, pure ObjectiveC project with one target |
| 2 | Dynamic frameworks separated from project target | Each framework has its own directory and dependencies |
| 3 | Dynamic frameworks aggregated into one project | One workspace with several framework projects and several application projects |

Let me expand second state as the most interesting state into another table. It considered that each framework is a project and must have its own directory, its one workspace and its own dependencies. Each framework completely separated from another framework or main application. I would like to integrate all of these targets  without `CocoaPods`, but it would be more complicated ( or not ).
However, `CocoaPods` exists in my project as a `CocoaLumberjack` provider and other third-party stuff.

| # | Project configuration name | Description | Integration into main target | Integration between frameworks | Integration into test target |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 2.1 | Test target alone | Test target is separated into different workspace, it also has its own dependencies | Archive binary framework | Archive binary framework | Archive binary framework |
| 2.2 | Test target and framework target together | Test target is copied into framework project. Now it shares all framework source code | Archive binary framework | Archive binary framework | Integrated automatically |

It would be nice if all of these things work. But I have custom setup and have caught many interesting pitfalls on this road. Let me show you common pitfalls and errors that you will catch.

### Hard part with maths
First of all, I will introduce variables.

* MT - Integration into main target
* TT - Integration into test target ( framework test target )
* FT - Integration into framework target
* FTT - Integration into framework test target ( another framework test target )

Last one is my favorite, it shows that you do something wrong. Possible values for each variable could be gathered into a table, lets see it. Empty value means that variable exhausts all possible values.

| Variable | Value # 1 | Value # 2 | Value # 3 |
| ---- | ---- | ---- | ---- |
| MT | Archive | CocoaPods | Source code |
| TT | Archive | CocoaPods | |
| FT | Archive | CocoaPods | |
| FTT | Archive | CocoaPods | |

We could change variables to more simple ones - union types. Let me introduce these variables.

* UFT - Union type of framework target and its test target.
* UFF - Union type of framework target and dependency framework target.
* UTTF - Union type of test target and dependency framework target.
* UMF - Union type of framework target and main application target.

Possible values for all these variables are:

| Variable | Value #1 | Value #2 | Value #3 |
| ---- | ---- | ---- | ---- |
| Union type | None | Workspace | Project |

Also, we have obvious rules that frameworks are less coupled than its test targets and test targets are less coupled then frameworks. If we add comparison operation between values of Union type, we have obvious coupling rule:

`None > Workspace > Project`

That means that items in different workspaces are less coupled than items in one workspace. Items in different workspaces could communicate between each other only in terms of binary or ready-to-use product. It is a fail.

After introducing this comparison, we have restriction on values of these variables that could be described as:

`UTTF >= UFF >= UFT`

We could assume that variables are equal, nothing wrong with it. Also, we could assume that main target is coupled with frameworks as test target of different framework coupled with dependency framework.

`UTTF =~ UMF`

They are nearly the same and could not be different.

The table of possible values of all variables:

| UFT | UFF | UTTF | UMF |
| --- | --- | --- | --- |
|N|N|N|N|
|W|N|N|N|
|W|W|N|N|
|W|W|W|W|
|P|N|N|N|
|P|W|N|N|
|P|W|W|W|
|P|P|N|N|
|P|P|W|W|
|P|P|P|P|

Let's check this table. UTTF should be equal UMF. Or check this code.

```ruby
[0, 1, 2]
.repeated_permutation(4).to_a
.select{|a| (a[0] <= a[1]) && (a[1] <= a[2]) && (a[2] == a[3])}
.map{|ar| ar.map{|a| case a; when 2; "N"; when 1; "W"; when 0; "P"; end }}
.map{|ar| "|" + (ar.join "|") + "|"}.reverse.join("\n")
```
No words about code quality, you could write it better, yes. But it works and it gives you this table.

Let me add values and description or names for these combinations.

|#| UFT | UFF | UTTF | UMF | Description |
| --- | --- | --- | --- | --- | --- |
|1|N|N|N|N|Workspace for each target (main, framework, test)|
|2|W|N|N|N|Framework and its test target in one workspace|
|3|W|W|N|N|All frameworks and all their test targets in one workspace|
|4|W|W|W|W|All targets in one workspace|
|5|P|N|N|N|Framework and its test target in one project. All frameworks in their workspaces|
|6|P|W|N|N|Framework and its test target in one project. All frameworks in one workspace. Main target in different workspace |
|7|P|W|W|W|Framework and its test target in one project. All targets in one workspace|
|8|P|P|N|N|All frameworks and all test targets in one project. Main target in different workspace|
|9|P|P|W|W|All frameworks and all test targets in one project. Main target and all frameworks projects in one workspace|
|10|P|P|P|P|All targets in one workspace|

Wow, the complex table.
What have I done in my project? I've jumped from the coupled #10 to the least coupled #1!

Was it good or bad?

## The pitfalls

I suppose that you would like to reread previous table. It is hard to describe possible values, I know. Also it is hard to understand all hidden features and pitfalls that each row represent.

At the end of the blog I will show my way in this journey from 10 to the last index that I've chosen.

### From 10 to 1

* Workspace for each target (main, framework, test)

I remember my first steps. I just tell myself: "Hey, separate whole project into frameworks, it would be awesome". Wrong.

#### The private pod. Simple case.

I create simple Network module that contain all network logic and also uses under the hood `AFNetworking`. Only one dependency - it is simple. I integrate this pod without any issues. Simple and obvious. Everything works fine.

I just tell myself: "Great stuff, dude! Let me test something!".

I add completely separate test target project with its own `CocoaPods` dependencies. It works as expected. I add `.podpsec` with `AFNetworking` dependency and its work great. No issues. Also, test target lives well in this case, it could install framework as dependency via `CocoaPods`.

We can conclude now that complete separation could be done via `CocoaPods` but is not recommended, because it is not natural that test for framework lives in completely different workspace.

#### The private pod. Complex case.

If you are addicted to dependencies manager, please, don't do it with your teammates.

For example, you have a `Database` framework with one dependency - `crypto_database_driver`. Everyone wants to secure user data.

I have done the same setup as for `Network`. Everything is fine. Except one thing. You could not add branch or commit as a dependency parameter in `.podspec`.

Again.

If you have a library that exists in `CocoaPods` repository and you want to add this library to your library as a dependency, you can not specify branch or commit, only tag.

It means that if this library has two versions, latest version eliminates one bug but adds another bug, you can't point to `bug-free` commit as a dependency in `.podspec` file.

That's it.

Now you can't use your library as `managed-by-cocoapods`.

#### The private pod. Unsolved case.

Further more, if you have a framework that uses another your framework, you may want to be smashed by train rather than solving this issue.

I have this setup. I am sorry.

I have setup in which three frameworks are in chain. Third framework depends on second and second framework depends on first. Each framework has its own test target that lives in separate workspace ( Remember that UFT is None ).

However, this setup could not be solved well in case of `@import` and framework search paths. I will leave it here as unsolved.

### From 1 to 2

* Framework and its test target in one workspace

Well, first step to go back is union of framework target and test target. Sure, it is nice setup but it doesn't solve many problems. One problem that it solves is [**Problem #1**][1]. It will have only one `Podfile` to manage both framework and test target for this framework.

Nice, but it is all.

### From 2 to 3

* All frameworks and all their test targets in one workspace

Curious setup. I also think about it, because now you could manage only one `Podfile` for all dependencies.

However, the main problem is `use_frameworks!` option. You still need to manage two `Podfile`s: one for this workspace with all frameworks and another for main target.

One advantage, that this setup do, is test target dependency. Not correct. Test target dependency on dependent framework. Still incorrect.

Now you could add other frameworks to specific test target and solve framework-dependency testability.

### From 3 to 4

* All targets in one workspace

I think about this setup as one the best setup. One of the biggest problems in this and previous setup ( all frameworks and test targets only in one workspace ) is [**Problem #2**][2]. Test target doesn't copy them. For that reason you should add `Copy frameworks build phase` and specify all dependent frameworks. 

### From 4 to 5

* Framework and its test target in one project. All frameworks in their workspaces

Nice setup, but it still has disadvantages. One of them is complex test target setup.

Yes, it lives near framework target, it has access to framework target code. But it doesn't solve [**Problem #2**][2] and you still need to add `Copy frameworks build phase` in test target. It simply doesn't copy them.

### From 5 to 6

* Framework and its test target in one project. All frameworks in one workspace. Main target in different workspace

Nice, very nice setup. You solve many problems from `CocoaPods` [**Problem #0**][3] to [**Problem #2**][2]. 

Very-very nice setup. 

However, how would you add your frameworks to your main target? Only one option - archive. 

But here we have another hidden pitfall. You should compile project for both platforms - simulator and iOS or arm and intel processors. Yes, architectures are different. For that reason you could write or use tool that `lipo`-ing two binaries into one fat framework. It is [**Problem #3**][3]. It is not solved automatically.

Also you may forget about bitcode. If it is enabled, you should also provide debug symbols in framework. It is [**Problem #4**][4]. It is not solved automatically.

### From 6 to 7

* Framework and its test target in one project. All targets in one workspace

I suppose that it one of the best setups. You have one workspace for all projects - so, you have only one `Podfile`. All dependencies are solved by it.

Whole project compiled from zero to one, [**Problem #4**][4] is eliminated. You don't need to provide fat framework to run application both on simulator and device - it's done already seamlessly. So, [**Problem #3**][3] is eliminated.

But one hidden problem still exists.

Suppose, that you have custom build configuration as I do. I duped it from Release and named it PublicBeta.

It exists only in main target project. It doesn't exists in frameworks target projects. It means that Xcode uses incorrect frameworks search paths to look for these frameworks.

To solve this problem you need to add build configuration to each framework project. It is a [**Problem #5**][5].

### From 7 to 8

* All frameworks and all test targets in one project. Main target in different workspace

It couples frameworks too tight into one project. However, it could solve one problem and it could not solve other problems.

It solves [**Problem #5**][5] by adding one build configuration into one frameworks project.

It doesn't solve other problems that reveal 6 setup - [**Problem #3**][3] and [**Problem #4**][4]. You still need to add these frameworks to main target as binaries.

### From 8 to 9

* All frameworks and all test targets in one project. Main target and all frameworks projects in one workspace

Well, this setup has all advantages of 7 setup. It also simplify [**Problem #5**][5] solution by adding only one configuration for frameworks project, not for each framework project.

However, one disadvantage is that an unicorn doesn't exist. You can't reuse frameworks in different projects because they are coupled too tight into one project.

### From 9 to 10

It is a common startup setup. It exists and hides nearly all hidden pitfalls.

# End?

I have an experience in solving all these complex setups. I've chosen for myself setup #7 as the best setup. It eliminates many problems and needs only `O(n) where n - count of frameworks` actions to solve [**Problem #5**][5].

## Problems and solutions

### Problem 0 Commit dependency
**Description**: CocoaPods does not allow you to specify commit-based dependency in your framework.  
**When**: If you have a framework that uses not mainstream latest library version and should use derivations of this library.  
Recommendation: Use private pods or don't use CocoaPods at all for framework handling.  
**Solution**: Eliminate CocoaPods from a project or add your framework alone as binary and install its dependency in main project from any commit.  
**Hint**: The linking phase will fix all problems and your project will have correct dependencies.  

### Problem 1 Podfiles duplicates
**Description**: You have two or more Podfiles to manage framework and test target for that framework.  
**When**: When they are not coupled too tight.  
**Recommendation**: They should be in one workspace or project.  

### Problem 2 Missing frameworks in test target
**Description**: Test target not runs in case of library image not found error.
**When**: When frameworks not coupled well and test target uses external framework as binary dependency.  
**Recommendation**: Put frameworks in one workspace.  
**Solution**: Add Copy frameworks build phase in test target to copy all dependent frameworks to Frameworks directory.  

### Problem 3 Fat framework
**Description**: You need a fat framework with simulator and device architectures.
**When**: It happens when you archive binary alone and put it into Link with Binary build phase in your test target.  
**Recommendation**: Try to avoid it.  
**Solution**: Build debug and release configurations binaries and `lipo`-ing them into one framework.

### Problem 4 Missing debug symbols problem
**Description**: You archive framework and add it to main target to build archive of application.  
**When**: It happens when you archive framework alone and later add it to main project.  
**Recommendation**: Try to avoid archive framework alone.  

### Problem 5 Missing Build configuration
**Description**: Projects has its own build configuration and doesn't share these configuration across workspace.  
**When**: It happens when you create one workspace for all targets and also add extra build configuration in your main target.  
**Recommendation**: Just know a solution.  
**Solution**: Add all build configurations from main target to each framework.

Good table with problems and solutions.

|#|Problem Name|Description|When|Recommendation|Solution|Hint|
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|0|Commit dependency|CocoaPods does not allow you to specify commit-based dependency in your framework.|If you have a framework that uses not mainstream latest library version and should use derivations of this library.|Use private pods or don't use CocoaPods at all for framework handling.|Eliminate CocoaPods from a project or add your framework alone as binary and install its dependency in main project from any commit.| The linking phase will fix all problems and your project will have correct dependencies.|
|1| Podfiles duplicates |You have two or more Podfiles to manage framework and test target for that framework.| When they are not coupled too tight. | They should be in one workspace or project. | They should be in one workspace or project. |
|2| Missing frameworks in test target | Test target not runs in case of library image not found error. | When frameworks not coupled well and test target uses external framework as binary dependency. | Put frameworks in one workspace. | Add Copy frameworks build phase in test target to copy all dependent frameworks to Frameworks directory. |
|3| Fat framework | You need a fat framework with simulator and device architectures. | It happens when you archive binary alone and put it into Link with Binary build phase in your test target. | Try to avoid it. | Build debug and release configurations binaries and `lipo`-ing them into one framework. |
|4| Missing debug symbols problem | You archive framework and add it to main target to build archive of application. | It happens when you archive framework alone and later add it to main project. | Try to avoid archive framework alone. |
|5| Missing Build configuration | Projects has its own build configuration and doesn't share these configuration across workspace. | It happens when you create one workspace for all targets and also add extra build configuration in your main target. | Just know a solution. | Add all build configurations from main target to each framework. |

[0]:#problem-0-commit-dependency
[1]:#problem-1-podfiles-duplicates
[2]:#problem-2-missing-frameworks-in-test-target
[3]:#problem-3-fat-framework
[4]:#problem-4-missing-debug-symbols
[5]:#problem-5-missing-build-configuration