---
id: 46
title: Using SQLite in your Windows 8 Metro style applications
date: 2012-06-29T21:59:58+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=46
permalink: /using-sqlite-in-your-windows-8-metro-style-applications/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Microsoft
  - SQLite
  - Windows 8
---
As a little surprise for developers, Windows 8 doesn’t come out with native database support. In the past months some SQLite portings came out, but none of the them was working really good. After the Windows Phone Summit (where Microsoft announced Windows Phone 8), the situation started to be more clear: SQLite will be the database officialy supported by Microsoft and SQLite will be officially released both for Windows 8 and Windows Phone 8. This way it’s likely to think that, as developers, we will be able to share not only the database but also the data access layer between the two platform: this will help us a lot porting our apps from one platform to the other.

Now SQLite’s branch for WinRT is available and we can start to use it to store the data of our applications, with the help of another library called **sqlite-net**, that provides LINQ based APIs to work with data, both with sync and async support.

In this post we’ll see how to start using SQLite in a XAML / C# application, how to create our first database and how to make some simple operations, like storing and reading data.

### Adding SQLite to our Metro style app

The first step is to download from the official <a href="http://sqlite.org/" target="_blank">SQLite</a> website the WinRT version: be careful that, since it’s a natice code library, you should get the specific version for the platform you’re going to support (x86 or x64). ARM is still not supported, but I expect it to be released soon.

For this example I’ve downloaded, from the <a href="http://sqlite.org/download.html" target="_blank">Download page</a>, the x86 version, which file name is� **sqlite-dll-winrt-x86-3071300.zip**

After you’ve downloaded it, you’ll have to extract the content somewhere and add the file **sqlite3.dll** to your project: be careful that, since the DLL it’s a native library that contains the SQLite engine (so it doesn’t provide any API to interact with it), you’ll simply have to copy it in the root of your project and make sure that the **Build Action** is set to **Content**.

Now that you have the engine you need something to interact with it: please welcome **sqlite-net**, a library available on NuGet and that supports WinRT, that provides data access APIs to interact with the database, using LINQ-based syntax.

Adding it is very simple: just right click on your project, choose **Manage NuGet packages**, search online for the package with name **sqlite-net** and install it. The package will add two classes in your project: **SQLIite.cs** (that provides sync access) and **SQLIteAsync.cs** (that provides, instead, asynchronous operations to interact with the database).

Now you’re ready to create your first database.

### 

### Create the database

The **sqlite-net** approach should be familiar to you if you’re a Windows Phone developer and you have already worked with SQL CE and native database support: we’ll have a class for every table that we want to create and we’re going to decorate our properties with some attributes, that will tell to the library how to generate them. We’ll use for this example a very simple class, that can be used to store a list of persons:

<pre class="brush: csharp;">public class Person
{
    [PrimaryKey, AutoIncrement]
    public int Id { get; set; }

    [MaxLength(30)]
    public string Name { get; set; }

    [MaxLength(30)]
    public string Surname { get; set; }

}</pre>

In this example you can see some of the simple attributes you can use to decorate your classes:

  * **PrimaryKey** is used to specify that the column will be the primary key of the table.
  * **AutoIncrement** usually is used in couple with a primary key; when this option is enabled the value of this column will be a number that will be automatically incremented every time a new row is inserted in the table.
  * **MaxLength** can be used with string properties to specify the maximum number of chars that will be stored in the column.

After you’ve defined the tables, it’s time to create the database. Since WinRT relies as much as possible on an asynchrnous pattern, we’ll use the async version of sqlite-net.

<pre class="brush: csharp;">private async void CreateDatabase()
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection("people");
    await conn.CreateTableAsync&lt;Person&gt;();
}</pre>

The first step is to create a **SQLiteAsynConnection** object, that identifies the connection to the database, like in the example: the parameter passed to the constructor is the name of the file that will be created in the local storage. Then we call the **CreateTableAsync<T>** method for every table that we want to create, where **T** is the type of data that we’re going to store in it (in the example, every row will be an element of the **Person** class). Notice that the method returns a **Task**, so we can use the keyword **await** to perform the operation asyncrhonously.

### 

### Play with the data

Now that we have a database, we can have some fun by adding and reading some data. Both operations arew very simple and, in both case, we’ll need a **SQLiteAsyncConnection** object that points to the same database.

To insert data we use the **InsertAsync** method, that simply accepts as parameter an istance of the object we’re going to save. Obviously, the object’s type should match the table’s type. Here is an example:

<pre class="brush: csharp;">private async void Button_Click_1(object sender, RoutedEventArgs e)
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection("people");

    Person person = new Person
    {
        Name = "Matteo",
        Surname = "Pagani"
    };

    await conn.InsertAsync(person);
}</pre>

To query the data, instead, we can access directly to the table using the **Table<T>** object: it supports LINQ queries, so we can simply use LINQ to search for the data we need. Then, we can call the **ToListAsync** method to get a **List<T>** of objects that matches the specified query. In the following example we look in the table **Person** all the users which name is Matteo and we print the results in the Output Window.

<pre class="brush: csharp;">private async void Button_Click_2(object sender, RoutedEventArgs e)
{
    SQLiteAsyncConnection conn = new SQLiteAsyncConnection("people");

    var query = conn.Table&lt;Person&gt;().Where(x =&gt; x.Name == "Matteo");
    var result = await query.ToListAsync();
    foreach (var item in result)
    {
        Debug.WriteLine(string.Format("{0}: {1} {2}", item.Id, item.Name, item.Surname));
    }
}</pre>

### 

### Where is my data?

If you want to take a look at your data, you can access to the path where Windows 8 stores the local storage of application. To find it, simply get the value of the property **Windows.Storage.ApplicationData.Current.LocalFolder.Path.** Once you have it, you’ll find in that folder a file with the same name that you’ve specified as parameter when you’ve created the **SQLiteAsyncConnection** object. If you want to open it, I suggest you to download and install an utility called **SQLite Database Browser**, that you can find at the website <http://sqlitebrowser.sourceforge.net/>.

With this utility you can open the database and explore it: you can see the tables, query the data and so on. Have fun!

**Update**: [here](http://qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/06/SqlLiteTest.zip) you can download a sample project