---
id: 4022
title: 'SkyDrive breaking changes with Live SDK: update your apps!'
date: 2013-07-17T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=4022
permalink: /skydrive-breaking-changes-with-live-sdk-update-your-apps/
categories:
  - Windows Phone
tags:
  - Skydrive
  - Windows Phone
---
**Update:�** as the developer Graham Haley has reported me on Twitter, Microsoft has fixed the server sode issue and [reported it in the official MSDN forum.](http://social.msdn.microsoft.com/Forums/live/en-US/52ab9b0f-71f5-437e-be0b-48115a9ba0a9/skydrive-update-related-to-sdk-5052-and-wlupload)� This means that your application should work fine now even if you&#8217;re using a Live SDK version prior to 5.4.

Unlike the other posts I usually write, this post will be very quick  <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" alt="Smile" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/07/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />Microsoft has introduced a breaking change in the SkyDrive APIs offered by the <a href="http://msdn.microsoft.com/en-us/live/ff621310.aspx" target="_blank">Live SDK</a> (which is the library, available on NuGet, that can be used to interact with Live services within a Windows Phone or Windows 8 application). If your application is using SkyDrive integration (like my app <a href="http://www.windowsphone.com/s?appid=a476ab62-44b4-4cca-9763-e7e064a1cc26" target="_blank">Movies Tracker</a>), for example, to provide backup and restore features, you may notice that the downloaded file is corrupted, so the restore procedure will always fail.

The solution is simple: update your project to use the latest Live SDK release, which is the **5.4**. Previous versions won’t work any more. The breaking change seems to be just server side: you don’t have to modify anything in your code. If you’re using NuGet, the operation is really simple: right click on your project, choose **Manage NuGet packages** and switch to the **Updates** tab. After a while, NuGet will find a new Live SDK version: just click on **Update** and you’re all set. Obviously, if your application is already using the 5.4 release you won’t have to do anything.

And if your application doesn’t support SkyDrive integration but you’re interested in implementing it, there’s a <a href="http://wp.qmatteoq.com/backup-and-restore-on-skydrive-in-windows-phone/" target="_blank">blog post I wrote</a> a while ago about this topic.

Happy development!