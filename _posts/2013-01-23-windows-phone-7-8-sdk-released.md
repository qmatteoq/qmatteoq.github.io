---
id: 1742
title: Windows Phone 7.8 SDK released
date: 2013-01-23T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1742
permalink: /windows-phone-7-8-sdk-released/
categories:
  - Windows Phone
tags:
  - Windows Phone
---
Yesterday evening Microsoft has released an update for the Windows Phone SDK to support Windows Phone 7.8, that is the new version of the OS that will be released soon for the devices of the previous generation. Since Windows Phone 8, due to the new kernel, isn’t available as update for the current devices, Microsoft has decided to release an “intermediate” version, that will introduce some of the new Windows Phone 8 features in the current generation’s devices. 

Most of the new features are for consumers: support to new tiles templates and sizes, Bing integration as a lock screen provider and so on. 

From a developer point of view, nothing changes: there are no new APIs or features that can be used by the developer. The only news is the support to new tiles templates and sizes also for third party applications. To use them, however, there aren’t new APIs, but you have to use reflection: the system’s DLL that is able to interact with the tiles is loaded at runtime and, if the application is running on a Windows Phone 7.8 device, you are able to use them to support the new tiles. 

The easiest way to take advantage of this feature is using Mangopollo, a library which <a href="http://wp.qmatteoq.com/how-to-support-windows-phone-8-devices-in-a-windows-phone-7-application/" target="_blank">I’ve talked you about in another post.</a> This library “hides” the reflection mechanism by offering to the developer a high level API, that works in the same way of the original APIs that are available in Windows Phone 8. 

Since there are no new APIs, which is the purpose of a new SDK? First, it’s not really a new SDK, but a patch, that is compatible both with the 7.1 SDK (if you don’t already have the 7.1.1 update, it’s included in this new release) and with the 8.0 SDK. This update will add to the existing list of available emulators two new images both based on Windows Phone 7.8: one standard and one with 256 MB of RAM, to simulate the low cost devices on the market. 

You can download the update from the following address: <http://www.microsoft.com/en-us/download/details.aspx?id=36474>