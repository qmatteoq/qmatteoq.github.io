---
id: 146
title: 'Windows 8 RTM is here: how to fix activation issues and submit apps to the store!'
date: 2012-08-16T10:00:30+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=146
permalink: /windows-8-rtm-is-here-how-to-fix-activation-issues-and-submit-apps-to-the-store/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Windows 8
---
Since yesterday, as previously announced, Windows 8 and Visual Studio 2012 RTM are available to MSDN and TechNet subscribers. Even if this is an important news, since it marks a new milestone for Microsoft, there isn’t too much to tell: thanks to the preview version released during this year we are already familiar with the new Microsoft OS and with the new way to develop applications using the new WinRT runtime.

In this post I would like just to highlight two important information: how to fix activation issues and how to start submitting apps to the store even if you don’t have a MSDN subscription.

### Activation issues

Many people (me included) have experienced issues while trying to activate their Windows 8 copy, by getting the error **DNS name** **does not exist.**� This problem occurs usually with the Enterprise edition, since it requires a multi activation key. There’s a workaround for this:

  1. Open a command prompt with admin privileges.
  2. Type **slmgr /ipk** followed by the product key you’ve obtained from MSDN (for example, slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx-xxxxx).
  3. Start the activation process.

This time everything should work fine and you’ll be able to enjoy all the Windows 8 features.

### Submit apps to the store

Now that the RTM is released developers will be able to submit apps to the store: the problem is that, obviously, you are required to compile and test your app against the RTM release but, if you don’t have a MSDN or TechNet subscription, you won’t be able to get a Windows 8 copy until 26th October.

To allow developers without a subscription to submit their apps anyway Microsoft has released an **evaluation version,** that will expire in 90 days after the installation. It’s a full Windows 8 version, so you can use for development purposes without any glitch.

The only downside is that the evaluation version **can’t be updated** to a full version after 26th October: you will need to install everything from scratch. For this reason, I suggest you to install it in a virtual machine or in a dedicated partition and not to use it as your main operating system. You can download the evaluation version <a href="http://msdn.microsoft.com/en-us/evalcenter/jj554510.aspx" target="_blank">from here</a>.

Have fun with Windows 8!