---
id: 1362
title: 'Lex.db: a new database storage solution for Windows Phone 8'
date: 2013-01-28T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1362
permalink: /lex-db-a-new-database-storage-solution-for-windows-phone-8/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Database
  - Windows 8
  - Windows Phone
---
Recently, on my blog, I’ve talked a lot about the database solutions that are available in Windows Phone 8. There are many different ways to manage database, each one with its pros and cons. SQL CE with LINQ to SQL is the most powerful one, but isn’t supported by Windows 8 and it’s hard to share the code and the database with other platforms. SQLite support is very promising and the engine is very fast, but at the moment it lacks a powerful library to manage data, so it’s a bit limited.

Please welcome a new participant in this “contest”: **Lex.db,** an interesting solution developed by Lex Lavnikov (pay often a visit <a href="http://lexblog.azurewebsites.net/" target="_blank">to his blog</a> because it regularly posts update about the project).**&nbsp;**Do you remember <a href="http://sterling.codeplex.com/" target="_blank">Sterling</a>? It’s an object oriented database that uses the file system to store data in a structured way, so that it can be used with LINQ to do queries and operation like a regular database. The biggest pro is that is available for many .NET technologies and it’s very fast, especially if you simply need to persist data: since it uses just the file system, it doesn’t add the overhead of a database engine.

**Lex.db** is based on the same principles, but it supports also Windows Runtime: for this reason it can be used not only with Windows Phone 8, but also with Windows Store apps and with the full .NET framework. How does it work? Lex.db is able to automatically serialize the data that we need to store in the file system and to keep in memory just the keys and the indexes that are needed to perform operations on the file. For this reason it’s really fast and often performances are superior than using SQL Lite.

Which are the downsides? For the moment, it still doesn’t support Windows Phone 7, so it’s not the perfect choice if you need to share the data layer of your application between two projects for Windows Phone 7 and Windows Phone 8. Plus, it’s not a real database solution: for example, for the moment relationships are not supported, so you’ll have to manually manage them if you need.

But, for example, if you’re going to develop a To Do List app (or any other app that needs to hold in storage a collection of data) and you want to share the data layer with a Windows Store version of the app it’s a really good solution.

Let’s see how to use it in a Windows Phone application (but the code would be exactly the same for a Windows Store app).

### The first setup

The simplest way to use Lex.Db in your project is to install it using NuGet: right click on your project, choose **Manage NuGet** packages and, using the built in search engine, look for and install the package called **Lex.db.**

After that we can create the entity that will hold the information that will be stored in the file system. Unlike with sqlite-net or LINQ to SQL, we don’t have to decorate classes with special attributes: we just need to define the class. Let’s use the ToDo List app example: in this case we need a class to store all the information about an activity.

<pre class="brush: csharp;">public class Activity
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public DateTime ExpireDate { get; set; }
}
</pre>

Now, exactly like we do with SQL CE or SQL Lite, we need to tell to the library which is the structure of our database. In our example we’ll have one “table” to store all the activities. First we declare a new **DbIstance** object in the page, so that can be used by every method of the class:

<pre class="brush: csharp;">private DbInstance db;

private async void LoadDatabase()
{
    db = new DbInstance("catalog");
    db.Map&lt;Activity&gt;()
      .Automap(x =&gt; x.Id);

    await db.InitializeAsync();
}
</pre>

Then we create a new instance of the **DbInstance** class, passing as parameter the name of the folder that will be created in the local storage to store all the data. We’ll need to refer always to this key in order to do operations on our data.

Then, using a fluent approach, we set which is the entity that we need to store with the **Map<T>** method, where T is the class we’re going to serialize (in our example, **Activity**). In the end we define which is the primary key of the table, that will be used to perform all the queries, using the **Automap** method: by using the lambda syntax we specify which property of our entity is the primary key, in our example the **Id** property. Before moving with doing operations we call the **InitializeAsync()** method on the **DbInstance** object, so that the library is able to create all the needed infrastructure to store the data. It’s an asynchronous operation, so we mark the method with the **async** keyword and we use the **await** keyword in order to wait that everything is correctly setup before doing something else. The library provides also synchronous methods, but I always suggest you to use the asynchronous ones, in order to provide a better user experience.

Now we are ready to save some data:

<pre class="brush: csharp;">private async void OnInsertDataClicked(object sender, RoutedEventArgs e)
{

    Activity activity = new Activity
                            {
                                Id = 1,
                                Title = "Activity",
                                Description = "This is an activity",
                                ExpireDate = DateTime.Now.AddDays(3)
                            };

    await db.Table&lt;Activity&gt;().SaveAsync(activity);
}
</pre>

The syntax is pretty straightforward: we create a new **Activity** object and we pass it to the **SaveAsync** method on the **Table<Activity>** object, which is the mapping with the “table” that stores our data. Simple, isn’t it? And if we want to get our data back? Here is an example:

<pre class="brush: csharp;">private async void OnReadDataClicked(object sender, RoutedEventArgs e)
{
    Activity[] activities = await db.Table&lt;Activity&gt;().LoadAllAsync();

    foreach (var activity in activities)
    {
        MessageBox.Show(activity.Title);
    }
}
</pre>

In this sample we retrieve, using an asynchronous method, all the **Activity** entities with the **LoadAllAsync** method, that is available using the **Table<T>** property of the database. Since the primary key of the table acts also as index, we can also query the table to get only the item with a specific id, by using the **LoadByKeyAsync** method:

<pre class="brush: csharp;">private async void OnReadDataClicked(object sender, RoutedEventArgs e)
{
    Activity activity = await db.Table&lt;Activity&gt;().LoadByKeyAsync(1);

    MessageBox.Show(activity.Title);
}</pre>

**Please note!** The **DbInstance** object exposes directly some methods to do these basics operations, like **Save()** and **LoadAll()**. The library is able to understand which class we are using and to do the operation on the correct table. Anyway, I suggest you to always access to the data using the **Table<T>** property because, this way, you have access also to the asynchronous method that, otherwise, wouldn’t be available.

### Using indexes

One common scenario in such an application is to query the data, to get only the items that meet specific criteria. This is what index are for: during the database setup you can choose to use one or more properties as index and use them to perform query operations. Let’s see an example: let’s say that we want to perform queries on the **Title** properties, to search for activities with a specific word in the title.

First we need to change the first database configuration:

<pre class="brush: csharp;">private async void LoadDatabase()
{
    db = new DbInstance("catalog");
    db.Map&lt;Activity&gt;()
      .Automap(x =&gt; x.Id)
      .WithIndex("Title", x =&gt; x.Title);

    await db.InitializeAsync();
}
</pre>

We’ve called a new method, which name is **WithIndex:** as parameter we pass the name of our index (**Title**, in our case) and, using a lambda expression, which is the property of our entity that will be mapped with the key.

Now we can execute queries on the **Title** property by using the same **LoadAllAsync** method we’ve seen before:

<pre class="brush: csharp;">private async void OnReadDataClicked(object sender, RoutedEventArgs e)
{
    List&lt;Activity&gt; activities = await db.Table&lt;Activity&gt;().LoadAllAsync("Title", "Activity");

    foreach (var activity in activities)
    {
        MessageBox.Show(activity.Description);
    }
}
</pre>

The difference with the previous sample is that now are passing two parameters to the **LoadAllAsync()** method: the first one is the name of the index to use, while the second one is the value we’re looking for. The result of this code is that the **activities** collection will contain all the items which **Title** property is equal to “Activity”.

### What’s going on under the hood?

If we want to see what’s happening under the hood we can use a tool like <a href="http://wptools.codeplex.com/" target="_blank">Windows Phone Power Tools</a> to explore the storage of our application. We’ll see that the library has created a folder called **Lex.db**. Inside it you’ll find a subfolder with the name of the database you have specified when you created the first **DbIstance** object: inside it you’ll find a .data file, which holds the real data, and a .key file, which is used for indexing. If you try open it with a text editor like Notepad you’ll see the content of the data you’ve stored in the application, plus some other special chars due to the encoding made by the serialization engine of the library.

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/image_thumb.png?resize=508%2C271" width="508" height="271"  data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/image.png)

If you want to experiment a bit with Lex.Db, use the following link to download a sample project I’ve created.

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:824f055b-5f8c-45ce-81de-6fa6ae1a00b4" class="wlWriterSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/01/LexDb.zip" target="_blank">Download the sample project</a>
  </p>
</div>