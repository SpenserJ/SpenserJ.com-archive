---
layout: post
title: "Building your first Adium Plugin"
date: 2010-08-13 16:51
comments: true
categories: 
- Adium
- Obj-C
- XCode
keywords: Adium, Plugin, XCode
---
**This guide is extremely outdated. It was originally authored on August 13th, 2010, and has not been updated recently.**

*As a warning, Adium Plugins use internal Adium APIs, which sometimes change. If you build a plugin, it may break with future versions of Adium, so be prepared for some upkeep down the road, unless you want to be rude and build a great program, and then abandon it and all of its users.* This guide assumes that you understand the basics of Objective-C, XCode, and programming in general. If you don't meet one of these prerequisites, please go research the basics and then come back.

<!-- more -->

## Step 1: Compiling Adium

Before you can start coding a plugin, you need the Adium plugin framework. To get this, download and build Adium from [http://trac.adium.im/wiki/GettingNewestAdiumSource](http://trac.adium.im/wiki/GettingNewestAdiumSource). Be sure to download the version that most of your plugin's users will be using (e.g. stable). If you cannot complete this step without googling, I would suggest learning the basics of Objective-C and XCode, among other things, first.

## Step 2: Creating your plugin's project

Open the `New Project` window, choose `Framework & Library`, choose `Bundle`, and set the Framework to `Cocoa`.
Right click on your Bundle under the `Targets` section of the sidebar, and choose `Get Info`. Go to the properties tab, and set the `Principle Class` to the same as your plugin name (I'm using MyFirstAdiumPlugin). Also, set your identifier, which is normally the reverse of your domain. Since my domain is SpenserJones.com, I will use com.spenserjones.MyFirstAdiumPlugin. Finally, set the `Creator` code to `AdIM`, and confirm that the Type is `BNDL`.
Under the Build tab of the Info window, change the following settings:
<table width="100%">
<tbody>
<tr><th style="text-align: center;">Setting</th><th style="text-align: center;">Value</th></tr>
<tr>
<td>Wrapper Extension</td>
<td>AdiumPlugin</td>
</tr>
<tr>
<td>Installation Directory</td>
<td>`$(HOME)/Library/Application Support/Adium 2.0/PlugIns/`</td>
</tr>
<tr>
<td>Base SDK</td>
<td>Mac OS X 10.5 (or the lowest version of Mac OS X you can set it to)</td>
</tr>
</tbody>
</table>
Finally, drag `$(ADIUM)/build/Development/Adium.framework` (where $(ADIUM) is the directory where you checked out Adium) into `Frameworks and Libraries/Other Frameworks` in the sidebar of XCode. When prompted, check the `Copy items into destination group's folder (if needed)` and `Recursively create groups for any added folders.`

## Step 3: Writing the code

Click on the folder `Classes` in the left pane of the main window of XCode, and then click `File - New File` (CMD-N). Choose `Cocoa Class` and then `Objective-C class`. Leave the subclass as `NSObject`, as we will be changing this shortly. Name the file the same name as your plugin, and check the `Also create \*.h` box.
Open up `MyFirstAdiumPlugin.h`, and add an import line of `#import <Adium/AIPlugin.h>` and also change `MyFirstAdiumPlugin : NSObject` to `MyFirstAdiumPlugin : NSObject <AIPlugin>`
Create two void functions in `MyFirstAdiumPlugin.m` called installPlugin and uninstallPlugin.  After doing these steps, your code should look like:
``` obj-c MyFirstAdiumPlugin.h
#import <Cocoa/Cocoa.h>
#import <Adium/AIPlugin.h>

@interface MyFirstAdiumPlugin : NSObject <AIPlugin> {
  

}

@end
```

``` obj-c MyFirstAdiumPlugin.m
#import "MyFirstAdiumPlugin.h"

@implementation MyFirstAdiumPlugin

- (void)installPlugin
{
  

}

- (void)uninstallPlugin
{


}

@end
```

Before you get too excited about having coded a plugin, there is one more step. Open `Info.plist` and add the following line, replacing `{VERSION}` with the version of Adium you compiled: `<key>AIMinimumAdiumVersionRequirement</key> <string>{VERSION}</string>`

As of writing this, the release version is 1.3, while the beta is 1.4. Adium 1.4 will require users to have Leopard or better, while 1.3 requires Tiger.

## Step 4: Bask in the glory of your first Adium plugin

All that is left is to compile and install your first Adium plugin. Click build, confirm that there are no errors, and then double-click `MyFirstAdiumPlugin/build/Debug/MyFirstAdiumPlugin.AdiumPlugin` from Finder.

## Step 5: Add some functionality

I'd assume that you want a plugin that actually DOES something, instead of sits pretty in your plugins folder, right? Adium's documentation for plugin creation is fairly sparse (and always has been), however there is a map of Adium's API at [http://trac.adium.im/wiki/MapOfAdium](http://trac.adium.im/wiki/MapOfAdium), and plenty of plugins online that you can examine. Hopefully I will find time soon to write up a continuation of this guide, for adding functionality. If you use this guide to create a plugin, I'd love it if you would share what you've done in the comments below!
