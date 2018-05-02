---
id: 582
title: 'Windows Phone and SQL Lite: what&rsquo;s the current situation?'
date: 2012-12-13T14:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=582
permalink: /windows-phone-and-sql-lite-whats-the-current-situation/
tags:
  - SQLite
  - Windows Phone
---
The last week I spoke at a Italian conference in Microsoft Italy called Windows Phone Developer Day about sharing code between Windows Phone and Windows 8. One of the topics was data access and how to share code to interact with a database between Windows Phone and Windows 8.

People was a bit confused and I got many questions because I told them that SQL Lite is supported by Windows Phone 8 but can’t be used at the moment. What does it mean?

Let’s make a step back: Windows Phone 7.5 has introduced support to relational databases thanks to SQL CE (as database engine) and LINQ to SQL (as manipulation library, to interact with the database using objects and classes instead of making queries). In Windows Phone 8 Microsoft has decided to go another way, in order to use a solution easier to port to Windows 8 and to other platforms: SQL Lite. In fact, this technology is cross platform and it’s available for Android, iOS, web application, client applications and much more. Probably you’ll have to rewrite the data access layer (due to the differences between technologies and languages), but you’ll be able to reuse the database.

Microsoft has worked with the team to provide an easy way to add SQL Lite to your Windows 8 and Windows Phone 8 project: the result is that, in the <a href="http://visualstudiogallery.msdn.microsoft.com/" target="_blank">Visual Studio Gallery</a>, you’ll find two extensions (one per platform) that can be used to add the SQL Lite engine to your project. The engine **is written in native code**: this means that you’ll need native code to interact with it and perform operations with the database.

And here comes the confusion I talked about in the beginning of the post: this extension allows you just to add the SQL Lite engine to your application, but doesn’t include a library to work with it using a high level language like C# or VB.NET. The problem is that, for the moment, this library doesn’t exist for Windows Phone 8: the result is that, even if you can add it, you don’t have a way to perform operations with a SQL Lite database, unless you write by yourself a wrapper to the native libraries.

Microsoft is committed into delivering a library as soon as possible, but at the moment we don’t have a release date yet.

On Windows 8 the situation is a bit different, because such libraries already exist: one of the most popular is **sqlite-net**, on which I talked about <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">in a previous post</a>. Unlucky, this library doesn’t work on Windows Phone 8, because it uses some features of the Windows Runtime that aren’t available in the Windows Phone subset.

In case you’re developing a Windows Phone application and you need to use a database, which are the possible solutions?

The first one is to keep using SQL CE, that works just fine in Windows Phone 8: it’s a very good solution in case you don’t need to port your application to Windows 8 or to another platform.

The second one is to use **csharp-sqlite**, that is a library published on <a href="http://code.google.com/p/csharp-sqlite/" target="_blank">Google Code</a> that allows to use the SQL Lite engine in a C# application. The library is supported by Windows Phone 8, so you can use it to reuse your SQL Lite database in a Windows Phone application. The downside? The C# library is based on code that is part of the .NET framework and that is not available in the Windows Runtime. This means that you’ll be able to reuse the database, but not the data access layer of your application.

And the third solution? Simply just wait that a native library for Windows Phone 8 will be available <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />