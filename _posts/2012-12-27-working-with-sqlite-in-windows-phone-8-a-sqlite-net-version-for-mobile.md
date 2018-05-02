---
id: 992
title: 'Working with SQLite in Windows Phone 8: a sqlite-net version for mobile'
date: 2012-12-27T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=992
permalink: /working-with-sqlite-in-windows-phone-8-a-sqlite-net-version-for-mobile/
categories:
  - Windows Phone
tags:
  - SQLite
  - Windows Phone
---
With a perfect timing, as soon as <a href="http://wp.qmatteoq.com/a-workaround-to-use-sqlite-in-a-windows-phone-8-application/" target="_blank">I’ve published my previous post</a> about using the **chsarp-sqlite** engine in combination with **sqlite-net,** Peter Huene has released a porting of the famous library for Windows Phone 8. What does it mean? That, finally, we are able to use the native SQLite engine that has been released as a Visual Studio extension and that we can use a common library to share our data layer with a Windows Store app for Windows 8.

At the moment, the project isn’t available on NuGet yet and requires two steps: the first one is to add a native class, that acts as a wrapper for the functions used by sqlite-net, and the second is to download a specific sqlite-net version, where the developer has replaced the usage of the csharp-sqlite engine with the native one.

Let’s start!

### Please welcome GitHub

Both projects are hosted on <a href="https://github.com/" target="_blank">GitHub</a> (a popular website to host open source projects that also acts as a source control system based on Git), so the best way to download and use both them is using Git: you can also download the project in a single zip file but, this way, every time the developer will change something you’ll have to download everything again and add the new project to your solution. If you’re not familiar with Git, the easiest way to use it is to download GitHub for Windows, which is a Windows client that is able to connect to repositories hosted on GitHub and to keep files in sync with the server.

Just download and install the application from <a href="http://windows.github.com/" target="_blank">here:</a> after you’ve launched it you’ll have to configure it for the first time. You’ll need to have a valid GitHub account: if you don’t have it, simply go to the website and create one. Once you’ve done you should see a window like this:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb5.png?resize=395%2C217" width="395" height="217"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image5.png)

&nbsp;

Unless you’ve already used GitHub and you already own one or more repositories, the window will be empty. Now go to the GitHub website and, specifically, to the **sqlite-net-wp8** repository, that is available at the URL [https://github.com/peterhuene/sqlite-net-wp8](https://github.com/peterhuene/sqlite-net-wp8 "https://github.com/peterhuene/sqlite-net-wp8"). At the top of the page, in the toolbar, you’ll find a button labeled **Clone in Windows**. Click on it and make sure that you’ve logged in in the website with the same credentials you used for the application, otherwise you’ll be redirected to the page to download the GitHub client.

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb6.png?resize=395%2C268" width="395" height="268"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image6.png)

Once you’ve done it the GitHub client will be opened and the repository will be automatically added to the local repositories list: after a while (the progress bar will show you the status of the operation) the whole repository will be downloaded in the default location, that is the folder _C:\Users\User\Documents\GitHub\_ (where **User** is your Windows username). Inside it you’ll find a folder called **sqlite-net-wp8:** that is the project that we need to add to our solution.

Since we’re already playing with GitHub, let’s download also the **sqlite-net** fork adapted to work with Windows Phone 8: repeat the operations we’ve just made on the repository available at the URL [https://github.com/peterhuene/sqlite-net](https://github.com/peterhuene/sqlite-net "https://github.com/peterhuene/sqlite-net").

The last thing to do is to make sure you’ve installed the **SQLite for Windows Phone** extension, that is available from the <a href="http://visualstudiogallery.msdn.microsoft.com/cd120b42-30f4-446e-8287-45387a4f40b7" target="_blank">Visual Studio Gallery</a>.

Now that we have everything we need, we can start working on our Windows Phone 8 project.

### Let’s play with SQLite

The first thing to do is to open Visual Studio 2012 and to create a **Windows Phone 8** application. Once you have it, it’s time to add to the solution the **sqlite-net-wp8** project we’ve downloaded from GitHub: simply right click on the solution, choose **Add existing project** and look for the file **Sqlite.vcxproj** in the sqlite-net-wp8 folder (that should be _C:\Users\User\Documents\GitHub\sqlite-net-wp8_). You’ll see the new project added in the Solution Explorer: it will have a different icon than the Windows Phone project, since it’s written in native code and not in C#.

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb7.png?resize=269%2C309" width="269" height="309"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image7.png)

As I’ve previously explained, this is just the native wrapper for some of the functions used by **sqlite-net:** now we need to add the real **sqlite-net** and we do that by simply copying the **Sqlite.cs** and **SqliteAsync.cs** files that are stored inside the **src** folder of the solution (that will be available, as for the other one, in the _C:\Usesr\User\Documents\GitHub_ folder) into our project. We can do that by simply right clicking on the Windows Phone project and choosing **Add existing item**.

Now we need to add a reference in our Windows Phone application both to the **sqlite-net-wp8** library and to the SQLite engine: right click on your project, choose **Add reference** and, in the **Solution** tab, look for the **sqlite** library; after that, look for the **SQLite for Windows Phone** library, that is available in the **Windows** **Phone** – **Extensions** section.

**UPDATE:** the developer, to keep supporting also the C# engine I’ve talked about <a href="http://wp.qmatteoq.com/a-workaround-to-use-sqlite-in-a-windows-phone-8-application/" target="_blank">in my previous post</a>, has recently added a new requirement to use his library; you’ll have to add a specific contitional build symbol, in order to properly use the native engine. To do that, right click on your project (the one that contains the **Sqlite.cs** and **SqliteAsync.cs** files you’ve previously added), choose **Properties**, click on the **Build** tab and, in the **Conditional compilation symbols** textbox add at the end the following symbol: **USE\_WP8\_NATIVE_SQLITE.** In a standard Windows Phone 8 project, you should have something like this:

_SILVERLIGHT;WINDOWS\_PHONE;USE\_WP8\_NATIVE\_SQLITE_

And now? Now we can simply copy and paste the code we’ve already seen in the <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">original post about Windows 8</a> or in <a href="http://wp.qmatteoq.com/a-workaround-to-use-sqlite-in-a-windows-phone-8-application/" target="_blank">the more recent post about csharp-sqlite</a>: since all these libraries are based on **sqlite-net**, the code needed to interact with the database and to create or read data will be exactly the same. Here are the usual examples I make about doing common operations:

**UPDATE**: as some readers have pointed out in the comments, with the previous code eveyrthing was working fine, but the database file was missing in the local storage. As the sqlite-net-wp8 developer pointed me out, there’s a difference between the Windows 8 and the Windows Phone version of the library. In Windows 8 you don’t have to set the path, it’s automatically created in the root of the local storage, unless you specify differently. In Windows Phone 8, instead, you have to pass the full path of the local storage where you want to create the databse: the code below has been updated to reflect this change.

&nbsp;

<pre class="brush: csharp;">//create the database
private async void CreateDatabase()
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection(Path.Combine(ApplicationData.Current.LocalFolder.Path, "people.db"), true);
    await conn.CreateTableAsync&lt;Person&gt;();
}

//insert some data
private async void Button_Click_1(object sender, RoutedEventArgs e)
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection(Path.Combine(ApplicationData.Current.LocalFolder.Path, "people.db"), true);

    Person person = new Person
    {
        Name = "Matteo",
        Surname = "Pagani"
    };

    await conn.InsertAsync(person);
}

//read the data
private async void Button_Click_2(object sender, RoutedEventArgs e)
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection(Path.Combine(ApplicationData.Current.LocalFolder.Path, "people.db"), true);

    var query = conn.Table&lt;Person&gt;().Where(x =&gt; x.Name == "Matteo");
    var result = await query.ToListAsync();
    foreach (var item in result)
    {
        Debug.WriteLine(string.Format("{0}: {1} {2}", item.Id, item.Name, item.Surname));
    }
}
</pre>

### 

### Be careful!

There are some things to keep in mind when you work with this library and SQLite. The first one is that, actually, both libraries are not available on NuGet: you’ll have to keep them updated by using GitHub for Windows and, from time to time, by syncing the repositories, in order to have your local copy updated with the changes. If you’re going to add the **sqlite-net-wp8** project to the solution, like I did in the post, you won’t have to do anything, you’ll just have to rebuild your project. In case of the **sqlite-net** fork, instead, since we’ve simply copied the files, you’ll need to overwrite the old ones with new ones, in case they are updated. Or, even better, you can add the two files as a link from the original project: this way you’ll simply have to update the libraries from GitHub to see the updates in your application.

The second important thing to consider is that the **sqlite-net-wp8** library is built against a specific SQLite version: if the SQLite team releases an update to the Visual Studio extension (so that Visual Studio is going to prompt you that there’s an update to install), don’t update it until the **sqlite-net-wp8** project has been updated. Otherwise, many references will be missing and you won’t be able to open the project at all.

Have fun!