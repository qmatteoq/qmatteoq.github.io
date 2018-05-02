---
id: 5442
title: New GDR3 emulators for Windows Phone Developers
date: 2014-01-07T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=5442
permalink: /new-gdr3-emulators-for-windows-phone-developers/
categories:
  - Windows Phone
tags:
  - Windows Phone
---
A few days ago Microsoft <a href="http://blogs.windows.com/windows_phone/b/wpdev/archive/2014/01/02/new-emulators-available-for-windows-phone-8-0-updates-2-and-3.aspx" target="_blank">has finally released a new set of emulators</a> that are aligned with the latest Windows Phone update, GDR3, which is available to developers since a while and that it’s going to be released soon for all the consumers. One of the most important additions is a new emulator image for the new resolution introduced with GDR3, which is 1080p (1080&#215;1920). But there’s an important thing to highlight: resolution and screen size are two different parameters! If a device is using the 1080p resolution, it doesn’t mean that it has a big screen. Take, as example, the Lumia 1320: it features the same resolution of other devices on the market (720p), like the HTC 8X, but it offers a much bigger screen (6 inches).

This means that knowing the device’s resolution isn’t enough: if you want to adapt your application to devices with big screens (by showing, for example, more information), you need to know the real size of the screen. All the emulators included in the SDK simulates a standard device: you can notice, in fact, that the 1080p emulator offers the standard interface with two columns of tiles, while devices like the Lumia 1520 and Lumia 1320 offers three columns of tiles.

And what if you want to optimize your app for bigger screens and you don’t have a real device to play with? Rudy Huyn, the developer behind many famous apps like 6tag or 6sec, has created a simple class that can be used to “fake” the big screen, so that you can test your apps using one of the standard emulators.

Here are two articles that you’ll find useful to adapt your applications:

  * [http://developer.nokia.com/Community/Wiki/Simulate\_1080p\_windows\_phone\_emulator](http://developer.nokia.com/Community/Wiki/Simulate_1080p_windows_phone_emulator) it’s the article by Rudy that explains how to create and use the fake class to simulate large screen devices 
      * <http://blogs.windows.com/windows_phone/b/wpdev/archive/2013/11/22/taking-advantage-of-large-screen-windows-phones.aspx> it’s the article by the Windows Phone team that offers some great tips on how you can adapt your application to take advantage of the bigger screens.</ul> 
    Happy coding!