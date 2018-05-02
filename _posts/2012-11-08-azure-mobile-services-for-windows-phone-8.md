---
id: 323
title: Azure Mobile Services for Windows Phone 8
date: 2012-11-08T10:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=323
permalink: /azure-mobile-services-for-windows-phone-8/
categories:
  - Windows Phone
tags:
  - Azure
  - Windows Phone
---
During the BUILD conference, right after the announcement of the public release of the Windows Phone 8 SDK, Microsoft has also announced, as expected, that Azure Mobile Services, that we’ve learned to use in the previous posts, are now compatible with Windows Phone 8.

Do you remember the tricky procedure that we had to implement <a href="http://wp.qmatteoq.com/having-fun-with-azure-mobile-services-the-windows-phone-application/" target="_blank">in this post</a> to manually interact with the server? If you’re going to develop an application for Windows Phone 8 you can forget it, since now the same great SDK we used to develop a Windows Store app is now available also for mobile. This means that we can use the same exact APIs and classes we’ve seen <a href="http://wp.qmatteoq.com/having-fun-with-azure-mobile-services-integrating-with-windows-8/" target="_blank">in this post</a>, without having to make HTTP requests to the server and manually parsing the JSON response.

The approach is exact the same: create an instance of the **MobileServiceClient** class (passing your service’s address and the secret key) and start doing operations using the available methods.

The only bad news? If your application is targeting Windows Phone 7, you still have to go with the manual approach, since the new SDK is based on WinRT and it’s compatible just with Windows Phone 8.

You can download the SDK from the following link: [https://go.microsoft.com/fwLink/?LinkID=268375&clcid=0x409](https://go.microsoft.com/fwLink/?LinkID=268375&clcid=0x409 "https://go.microsoft.com/fwLink/?LinkID=268375&clcid=0x409")