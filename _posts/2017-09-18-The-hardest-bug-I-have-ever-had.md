---
layout: post
title: "The hardest bug I have ever had."
date: 2017-09-18 16:16:01 +0300
categories: localization resources build-configurations
---

Today I would like to answer a simple question: "Why my application doesn't have app icon after build?".  
I promise you that you are free from this bug today and forever.  
I am anxious about future Xcode updates.  
Thus, I posted this bug to Apple Bug Report and attached sample project.  
It is disappeared in Xcode 9.

However, let's start to catch it.

# Long time ago.

I asked myself: "How could I determine an app build configuration at a glance?".  
It would be nice if you just look at a phone screen and know everything about  a build configuration.

- Is it Debug? 
- Is it Release? 
- Is it even PublicBeta?
- Is it whatever configuration you imagine?

# Different Icons and App Names.
After googling I found a good solution - [build configurations and assets customizations](https://engineering.circle.com/different-app-icons-for-your-ios-beta-dev-and-release-builds-af4d209cdbfd).  

What am I talking about?  
`.xcconfig` files could make everything a bit flexible.  
They are so cute that you can't figure out what to do with them.  
They don't have [complete or incomplete documentation](https://pewpewthespells.com/blog/xcconfig_guide.html) and their syntax too simple to be ambiguous.  

I defined my own settings in build settings tab.  
You could choose app name depending on your build configuration.  

# Example App.
I will start with application Nexus. Name dedicated to trilogy Rosy Cruxifiction by Henry Miller.  
Nexus is a third book in trilogy. It is a release book in terms of deployment.   
A public beta book is a second book Plexus.
A development book is a first book Sexus.

Now, let's think about app names on phone screen.  
You could do it in various ways.  

But for simplicity I will choose `Build Configurations` and various `Schemes`.  
I need several build configurations for releases - one for Testflight and one 
for App Store.  

So, I dup Release build configuration and rename it to Public Beta.  
The main reason for that - users would like to install beautiful app from App Store and tests would like to install fat functional app from TestFlight.  
Beta testers can provide more complex app analysis by using Tester toolkit that I shipped in TestFlight build.  
For example, I would help testers to switch servers from Test to Public Beta or even to Production.  

I want to help testers to determine in which configuration (Debug / PublicBeta / Release) bug happens.  

For this reason I define Product Name and Assets AppIcons name to switch between app icons in build configurations.  

|Configuration Name|Product Name|App Icon Color|Build Destination|
|---|---|---|---|
|Debug|Sexus|Red|Developer Phone|
|PublicBeta|Plexus|Green|Testflight|
|Release|Nexus|Blue|AppStore|


And everything works fine.  
You create three different build configurations for a target with two different schemes (PublicBeta and Release) on Archive action.  

Simple and curious.  
You have what you want.  
App with different Schemes for each purpose - Beta testers and users.

# Localizations for images and pictures.
Another simple idea - localizations for pictures or assets.  
You have an image with a  (for example, an image with a specific localizable string).  
Your main goal - localization of these images.  
You should put asset folders in different lang.lproj folders as you do with `.strings` files.  

Asset catalog is a resource? Of course.  
Could it be treated as a localized resource? Pretty sure, yes.

After grouping all necessary images into assets folder ( I choose `LocalizedInterfaces.xcassets` ), you will have project structure.


- ```en.lproj/LocalizedInterfaces.xcassets```
- ```ru.lproj/LocalizedInterfaces.xcassets```
- ```fr.lproj/LocalizedInterfaces.xcassets```

Not bad, right?

# Two ideas - one hidden trouble.  
General workflow hides everything. You don't see a trouble before some refactoring or `Delete-all add-all` action.  
Yes, I do it sometimes - deleting all folders in Xcode project and re-adding them.  
I don't appreciate this technique, however, it is necessary to keep everything in alphabetical order - both, file system folders and Xcode groups.

# The Bug Appears

Resources are splitted into different folders.

- ```Interfaces.xcassets``` folder contains all small icons for an app.  
- ```AppIcons.appicons``` folder contains AppIcons.
- ```LocalizedInterfaces``` folder contains localized images.

Build configurations are these.

- ```Debug```
- ```PublicBeta```
- ```Release```

```AppIcons.xcassets``` has folders.

|Configuration Name|AppIcon.appiconset|
|---|---|
|Debug|AppIcon_Debug.appiconset|
|PublicBeta|AppIcon_PublicBeta.appiconset|
|Release|AppIcon.appiconset|

This setup works like a charm, everything is nice before The Bug Appears.

# The Road to Bug

One day I deleted all and added all files in project.  
I hit ```Cmd + R``` and waited for a build.

Project was good, build was good, app without app icon was...

You know all words that I could say in this situation.

> The project has two year history and somehow bug happens now?  
> What a nice bug could appear?

Besides, it was a day of release when I should create screenshots for AppStore.

I was a bad man if I said that Xcode didn't work properly.  
Well, it did, but one more prerequisite is a complex project structure.

I was incorrect in bug steps - they are more curious.

Let me reveal hidden steps.

First of all, I have several targets: core target and extra target. Extra target has all features of Core target except some resources.  

So, my release was related to all targets - Core target and extra target.
Core target feels good after `Delete-all add-all` action. Extra target doesn't.

Let's see what happens in detail.

I have extra target with `Custom` folder. Here is a structure of folders:



```
# Project folder
Core
|-- Resources
	|-- Images
		|-- AppIcon.xcassets
			|-- AppIcon_Debug.appiconset
			|-- AppIcon_PublicBeta.appiconset
			|-- AppIcon_Release.appiconset
		|-- Interfaces.xcassets
		|-- LocalizedInterfaces.xcassets
		
ExtraTarget
|-- Custom
	|-- Resources
		|-- Images
			|-- AppIcon.xcassets
				|-- AppIcon_Debug.appiconset
				|-- AppIcon_PublicBeta.appiconset
				|-- AppIcon_Release.appiconset
			|-- CustomInterfaces.xcassets
```

It seems reasonable to keep extra target resources in target subfolder. Yes, Core resources could be placed somewhere else, but why not? It was done like that.

`Delete-all add-all` action in extra target's case did additional thing - it adds default AppIcon asset. After that all default resources should be removed.

It sounds nice, right? So think I.   
However, an accident happened.

I selected Core folder and Custom folder.

# Down the Rabbit-Hole.

I deleted both folders, yes, accident happens. Nothing bad, re-adding them should fix everything. I removed unnecessary default resources.

Build started, but app icon disappeared. I checked everything again. Rerun. The same.

> Did I ever tell you what the definition of insanity is? Insanity is doing the exact... same fucking thing... over and over again expecting... shit to change... That. Is. Crazy. (c) Vaas Montenegro, Far Cry 3

Ok, let's delete all custom resources and re-add them again.

The situation became worse. All custom interfaces disappeared.  
I checked everything again. All things seem fine.

Rerun. The same. 

> Either the well was very deep, or she fell very slowly, for she had plenty of time as she went down to look about her and to wonder what was going to happen next. (c) Alice's Adventures in Wonderland, Lewis Carroll

I checked copy files build phase. Everything is fine.

Rerun. The same.

I opened archive folder and ended my fall. Feet feel ground.

# The dragon

Did you ever try to inspect app archive? I assure you that it is funny. You will see all resources that would be delivered with app binary.

In general situation `.xcarchive` contains `.app` folder which contains all app data: binary, compiled assets, nibs, localized strings and also app icons which a spreaded into .app without grouping in subfolder. Localized resources are grouped in `lang.lproj` folders.

I opened `.app` folder. It was corrupted. It didn't contain app icons at all. No icons. They are anywhere but not in folder. I inspected subfolders and found app icons in one of `lang.lproj` subfolder. How could they be in localized subfolder? They should be in the `.app` folder.

What was wrong?

# The head of the dragon

In general, you don't touch assets. They live somewhere inside your application. You only care about new images. Xcode compiles them and delivers to `.app` folder for you.

I started inspecting Copy Files build phase - the only place where any clues or evidences could be.

I noticed something interesting.

`AppIcon.xcassets`, `CustomInterfaces.xcassets` were not present in app. And also they were below `LocalizedInterfaces.xcassets`.

I moved down `LocalizedInterfaces.xcassets` to the end of the Copy Files Build phase files list.

`Cmd + R`

# The end

App icon appeared on the phone screen.  
I didn't believe it. I opened `.app` and assured myself that app icons were in this folder. They were in the root of it.