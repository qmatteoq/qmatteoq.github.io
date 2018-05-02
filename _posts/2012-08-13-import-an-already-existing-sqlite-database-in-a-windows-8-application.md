---
id: 123
title: Import an already existing SQLite database in a Windows 8 application
date: 2012-08-13T10:00:46+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=123
permalink: /import-an-already-existing-sqlite-database-in-a-windows-8-application/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - SQLite
  - Windows 8
---
One of the comments I’ve received <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">in the blog post I wrote about using SQLite in a Windows 8 application</a> was a really interesting question: what can I do if I have a pre populated SQLite database and I want to use it in my application? In fact, the blog post I wrote was based on a “code first” approach: according to some classes attributes, the database was generated during the first execution of the application.

The solution is pretty simple, thanks to the WinRTs APIs used to interact with the storage. If you recall the explanation in my previous post, you’ll know that **sqlite-net** (the library used to interact with the SQLite database using a LINQ syntax) looks for the database file inside the local folder of the application. What we’re going to do is to include our database file into the project and then, when the application is launched for the first time, copy it into the local storage, so that the library can access to it.

The first step is to copy your database in to the Visual Studio project and, from the **Properties** window, set the **Build action�** to **Content**.

Once you’ve done this operation, you’ll be able to access to the files embedded in your project thanks to the **Package.Current**.**InstalledLocation** object that is available in the **Windows.ApplicationModel** namespace.

The **InstalledLocation’**s type is **StorageFolder**, which is the base class of all the folders mapping in WinRT: for this reason, it exposes all the standard methods to interact with the storage, like getting a file or a folder. This way we can use the **GetFileAsync** method to get a reference to the database embedded into the project and, after that, using the **CopyAsync** method we can copy it into the local storage of the application. We can copy it in the root of the local storage (like in the following example) or in a specific folder, by getting a reference to it first using the **GetFolderAsync** method.

Here is a sample code:

<pre class="brush: csharp;">private async Task CopyDatabase()
{
    bool isDatabaseExisting = false;

    try
    {
        StorageFile storageFile = await ApplicationData.Current.LocalFolder.GetFileAsync("people.db");
        isDatabaseExisting = true;
    }
    catch
    {
        isDatabaseExisting = false;
    }

    if (!isDatabaseExisting)
    {
        StorageFile databaseFile = await Package.Current.InstalledLocation.GetFileAsync("people.db");
        await databaseFile.CopyAsync(ApplicationData.Current.LocalFolder);
    }
}</pre>

&nbsp;

The code should be simple to understand: the first thing we do is to check if the file already exists or not. If it doesn’t exist, we get a reference to the file embedded in the Visual Studio project (called **people.db**) and then we copy it in the local storage (that is mapped with the object **ApplicationData.Current.LocalFolder**). This way the file is copied in the root of the local storage.

After that, we can use the same code we’ve seen in the first post to access to the database and perform queries: it’s important to remember that the name of the classes that identify the database tables should have the same name of the tables in the database we’ve imported.