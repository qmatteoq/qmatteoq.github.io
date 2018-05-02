---
id: 902
title: A workaround to use SQLite in a Windows Phone 8 application
date: 2012-12-21T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=902
permalink: /a-workaround-to-use-sqlite-in-a-windows-phone-8-application/
categories:
  - Windows Phone
tags:
  - SQLite
  - Windows Phone
---
Recently <a href="http://wp.qmatteoq.com/windows-phone-and-sql-lite-whats-the-current-situation/" target="_blank">I’ve made a post</a> where I’ve tried to explain better the SQL Lite support situation on Windows Phone 8. The core of the post was very simple: the SQLite engine is available for Windows Phone 8, but actually a library to manipulate data using a high level language like C# or VB.NET is missing.

One of the workarounds I’ve talked about was using **csharp-sqlite**, an open source project hosted on <a href="http://code.google.com/p/csharp-sqlite/" target="_blank">Google Code</a> that embeds the SQLite engine into a C# library, so that it can be used also by applications based on the .NET framework. This project also features a client library, that can be used to manipulate the database: there’s also a Windows Phone porting of the library, but you can’t share it with a Windows 8 project due to the lack of support to the .NET framework.

Luckily, there’s still a solution to share your data layer with a Windows 8 application: by using **sqlite-net.** I’ve already talked about this library <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">in a previous post</a> and this is, probably, the most famous wrapper for the native SQLite engine. The good news is that sqlite-net supports two different SQLite engines, according to the platform: the native one on Windows 8 and the **csharp-sqlite** one on Windows Phone 7 or 8. The result is that that you’ll be able to use the sqlite-net classes and methods on both platforms and share the code between them: according to the platform, the proper SQLite engine will be used. Is there a downside? Yes, **csharp-sqlite** is a porting and is not managed directly by the SQLite team. This means that every time the SQLite team releases an update of the engine the C# engine should be recompiled against the new one. Plus, it seems that the project has been abandoned: the latest version has been released on 26th August 2011, so it’s a little bit old. This means that you’re going to miss all the improvements that the SQLite team has made in the last year.

But, since there are no other options at the moment, it worth a try to take a closer look to this option, especially if you just have to store some data and you don’t have a complex database to manage.

### 

### Grabbing the engine

The first thing to do is to go the Download page of the **csharp-sqlite** project and download the latest available version, that is the <a href="http://code.google.com/p/csharp-sqlite/downloads/detail?name=csharp-sqlite_3_7_7_1_71.zip&#038;can=2&#038;q=" target="_blank">3.7.7.1.71</a>. There isn’t a binary version or a package on NuGet: you’ll have to manually open the solution file **Community.CsharpSqlite.SQLiteClient.WP.sln** inside the Community.CsharpSqlite.SQLiteClient.WP folder. Visual Studio 2012 will update some files of the solutions, since it has been created using Visual Studio 2010. Now build the solution: you may get some compilation errors, in this case try to build just the **Community.CsharpSqlite.WinPhone** project; in fact, the build errors are just caused by some unit tests that are not using valid code.

Once you’ve done that, open the project’s folder and grab the **Community.CsharpSqlite.WinPhone.dll** file from the **bin/Debug** folder: it’s the reference you’ll need to add to your Windows Phone 8 project, so go and create a new one using the basic **Windows Phone application** template and choose Windows Phone OS 8.0 as target OS. Now you’ll need to add a reference to the library we’ve copied before, by right clicking on the project and choosing **Add reference**. After you’ve done that, it’s time to install **sqlite-net** by using **NuGet**: right click again on the project, choose **Manage NuGet** **packages**, look for and install the package called **sqlite-net.**

### Playing with SQLite

Once you’ve done, you’ll find some new files in your project: open the **SQLite.cs** file; you’ll see, right at the top, what I was talking about at the beginning at the post.

<pre class="brush: csharp;">#if WINDOWS_PHONE
    #define USE_CSHARP_SQLITE
#endif

#if USE_CSHARP_SQLITE
    using Community.CsharpSqlite;
    using Sqlite3DatabaseHandle = Community.CsharpSqlite.Sqlite3.sqlite3;
    using Sqlite3Statement = Community.CsharpSqlite.Sqlite3.Vdbe;
#else
    using Sqlite3DatabaseHandle = System.IntPtr;
    using Sqlite3Statement = System.IntPtr;
#endif
</pre>

As you can see, by using conditional compilation instructions, sqlite-net will define and activate the symbol **USE\_CSHARP\_SQLITE** in case the app is running on Windows Phone. This symbol will be used across the library to wrap the different behaviors to the standard sqlite-net methods and classes: this way, you’ll be able to use the same code to read and write data to the database, regardless of the platform.

And now what? You can copy and paste the same code I’ve explained <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">in this post</a>: since the library is the same, you’ll be able to use the same approach to work with the data. For example, you can do something like this to create the database and a table to store a **Person** entity:

<pre class="brush: csharp;">private async void CreateDatabase()
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection("people");
    await conn.CreateTableAsync&lt;Person>();
}
</pre>

After that, you can insert some data in the database:

<pre class="brush: csharp;">private async void Button_Click_1(object sender, RoutedEventArgs e)
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection("people");

    Person person = new Person
    {
        Name = "Matteo",
        Surname = "Pagani"
    };

    await conn.InsertAsync(person);
}
</pre>

or read the same data and write it into the Output Window:

<pre class="brush: csharp;">private async void Button_Click_2(object sender, RoutedEventArgs e)
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection("people");

    var query = conn.Table&lt;Person>().Where(x => x.Name == "Matteo");
    var result = await query.ToListAsync();
    foreach (var item in result)
    {
        Debug.WriteLine(string.Format("{0}: {1} {2}", item.Id, item.Name, item.Surname));
    }
}
</pre>

If I would have isolated the code to interact with the database in another class (for example, a service class) I would have been able to use it both in a Windows 8 and a Windows Phone 8 application, without changing it: **sqlite-net** would have done all the dirty work of using the correct engine for me.

### Conclusion

In this post we’ve seen a way to interact with a SQLite database and share the code between Windows Phone and Windows 8. It’s not the best approach, because it’s forcing us to use an unofficial porting of SQLite, that isn’t updated since more than a year. But, if you don’t have a complex scenario and you need to start working on your application as soon as possible and you need to share code with Windows 8 (so SQL CE it’s not a solution), it can be an acceptable approach. Obviously, the best solution would be to have a native **sqlite-net** implementation also for Windows Phone 8, in order to use the native SQLite engine that is already available in Windows Phone 8. It will come… eventually, one day or the other <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none;border-left-style: none;border-bottom-style: none;border-right-style: none" alt="Smile" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/wlEmoticon-smile1.png?w=640" data-recalc-dims="1" />

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:a20be5d6-9da0-4c9e-be89-fb9ea19250eb" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2012/12/SqlLiteWP8.zip" target="_blank">Download the sample project</a>
  </p>
</div>