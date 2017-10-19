---
layout: post
title: "The hardest bug I have ever had."
date: 2017-09-18 16:16:01 -0600
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
General workflow hides everything. You don't see a trouble before some refactoring or "delete all - add all" action.  
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

# The Bug Steps

One day I deleted all and added all files in project.  
I hit ```Cmd + R``` and waited for a build.

Project was good, build was good, app without app icon was...

You know all words that I could say in this situation.

> The project has two year history. And somehow bug happens now?  
> What a nice bug could appear?

Besides, it was a day of release when I should create screenshots for AppStore.

# One head of a dragon

I was a bad man if I said that Xcode didn't work properly.

Well, it did, but one more prerequisite is a complex project structure.

I was incorrect in bug steps - they are more curious.