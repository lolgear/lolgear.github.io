---
layout: post
title: "The hardest bug I have ever had."
date: 2017-09-18 16:16:01 -0600
categories: jekyll update
---

Today I would like to answer simple question: "Why my application doesn't have app icon after build?".
I promise you that you are free from this bug today and forever.
I am anxious about future Xcode updates.
Thus, I posted this bug to Apple Bug Report and attached sample project.

However, let's start to catch it.

# Long time ago.

I asked myself: "How could I determine app build configuration at glance?".  
It would be nice if you just look at phone screen and know everything about build configuration.

- Is it Debug? 
- Is it Release? 
- Is it even PublicBeta?

# Different Icons and App Names.
After googling I found a good solution - [build configurations and assets customizations](https://engineering.circle.com/different-app-icons-for-your-ios-beta-dev-and-release-builds-af4d209cdbfd).  

What am I talking about?  
`xcconfig` files could make everything a bit flexible.  
They are so cute that you can't figure out what to do with them.  
They don't have [normal documentation](https://pewpewthespells.com/blog/xcconfig_guide.html) and their syntax too simple to be ambiguous.  

I defined my own settings in build settings tab.  
It is so gorgeous! 
You could name app depending on your build configuraiton.  

# Example App.
Ok, let's, for example, assume application Nexus.  
No, I don't mean I like nexuses or even so. I mean 'Nexus' app due to bibliography of American writer Henry Miller. Also, I need a trilogy.  
Nexus is a third book in trilogy. It is release book in terms of deployment.  
For public beta I will choose second book - Plexus.  
And for myself and for a team I will choose first book - Sexus.  

Now, let's think about app names on phone screen.  
You could do it in various ways.  
But for simplicity and reasonablity I will choose Build Configurations and various Schemes.  
I need several build configurations for release - one for Testflight and one for App Store.  
So, I dup Release build configuration and rename it as Public Beta.  
The main reason for that - people would like to install beautiful app from App Store and fat functional app form TestFlight.  
Beta testers can provide more complex app analysis by using Tester toolkit that I shipped in TestFlight build.  
For example, I would help my testers to switch servers from Test to Public Beta or even Production.  
I want to help users to tell me in which configuration (Debug / PublicBeta / Release) bug happens.  

For that reason I define Product Name and Assets app icons name.  

- Debug - Sexus, red icon.
- PublicBeta - Plexus, green icon.
- Release - Nexus, blue icon.

And everything works fine.
You just create three different build configurations for ONE target with two different schemes (PublicBeta and Release).  
Simple and curious. And you have what you want. App with different Schemes for each purpose - Beta testers and users.

# Localizations for images and pictures.
Another simple idea - localizations for pictures or assets.  
You have words in picture (for example, image with name of specific status).  
And you want different pictures for different languages.  
So, as you know basic localization, you should put asset folders in different code.lproj.  
And what you have in the end?  

- ```en.lproj/LocalizedInterfaces.xcassets```
- ```ru.lproj/LocalizedInterfaces.xcassets```

# Two ideas - one hidden trouble.  
General workflow hides everything. You don't see trouble before some refactoring or delete all - add all action.  
Yes, I do it sometimes - deleting all folders in Xcode project file and re-adding them.  
I don't appreciate this technique, however, it is necessary to keep everything in alphabetical order - both, folder in file system and abstract folder in Xcode project.

And here it is.
I have big 
- ```Interfaces.xcassets``` folder contains all small icons for app.  
- ```AppIcons.appicons``` folder contains AppIcons for Release, DebugAppIcons for Debug, PublicBetaAppIcons for PublicBeta.
- ```LocalizedInterfaces``` folder for localized pictures.