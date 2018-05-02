---
id: 3492
title: 'A new SQLite wrapper for Windows Phone 8 and Windows 8 &ndash; The basics'
date: 2013-06-04T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3492
permalink: /a-new-sqlite-wrapper-for-windows-phone-8-and-windows-8-the-basics/
categories:
  - Windows 8
  - Windows Phone
tags:
  - SQLite
  - Windows 8
  - Windows Phone
---
One of the topics that gathered more attention on my blog is SQLite: developing an application without the need to store, somehow, the data locally is almost impossible, so developers are always searching the best way to achieve this objective. SQLite is one of the most interesting solutions out there, since it’s open source and it’s available for almost every platform on the market (mobile, web, client, etc.).

<a href="http://wp.qmatteoq.com/working-with-sqlite-in-windows-phone-8-a-sqlite-net-version-for-mobile/" target="_blank">I’ve already talked you about sqlite-net</a>, a library that is available for Windows Phone 8 and Windows 8 that can be used to perform operations on a SQLite database using a high level API that support LINQ operations. This library is very simple to use, but it has some limitations: the biggest one is that doesn’t support relationships out of the box.

Now Peter Torr (a member of the Windows Phone development team in Microsoft) with the support of Andy Wigley (a former Windows Phone Development MVP that now has joined Microsoft) have released on Codeplex a new SQLite wrapper, that is totally different from sqlite-net and that satisfies another type of approach: total control. In fact, this new wrapper doesn’t support any LINQ operation but only manual SQL statement: the biggest pro is that you can perform any type of operation, even managing relationships. The biggest cons is that you’ll have to rely on old plain SQL queries to do any operation, even the simplest ones, like creating a table or inserting some data.

Let’s see, step by step, how to use it and how the approach is different from using sqlite-net. In this first post we’re going to see the basics, in the next one we’ll talk about relationships.

### Configure the wrapper

For the moment, since it’s written in native code, the library isn’t available on NuGet: you’ll have to download the source code <a href="http://sqlwinrt.codeplex.com/" target="_blank">from the Codeplex page</a>. The solution contains two projects:� one is called **SQLiteWinRT** and the other one is called **SQLiteWinRTPhone.** The first one is for Windows Store apps, the second one for Windows Phone 8 apps (Windows Phone 7 is not supported, since it doesn’t support native code): you’ll have to add to your solution the project that fits your needs. For this tutorial I’m going to create a Windows Phone 8 application, so I’m going to use the second project. Plus, as we did for sqlite-net, we need to install the official SQLite runtime, that is available as a Visual Studio extension: here is the <a href="http://visualstudiogallery.msdn.microsoft.com/23f6c55a-4909-4b1f-80b1-25792b11639e" target="_blank">link</a> for the Windows Store apps version, here, instead, is the <a href="http://visualstudiogallery.msdn.microsoft.com/cd120b42-30f4-446e-8287-45387a4f40b7" target="_blank">link</a> for the Windows Phone 8 version.

Now you’re ready to use it! We’re going to use the same sample we used in the sqlite-net post: a table to store data about people.

### 

### Create the database

Let’s see how to do it:

<pre class="brush: csharp;">private async void OnCreateDatabaseClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "CREATE TABLE PEOPLE " +
                   "(Id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL," +
                   "Name varchar(100), " +
                   "Surname varchar(100))";

    await database.ExecuteStatementAsync(query);
}</pre>

First we create a new **Database** object, that we’re going to use to make operations on the database. There are various ways to instantiate it: in this sample we’re going to create it by specifying the folder in the isolated storage where to save the file and the file name. The folder is passed to the constructor using a **StorageFolder** object, which is the Windows Runtime class that identifies a folder in the isolated storage. In the sample, we’re simply using the storage’s root. If the database doesn’t exist, it will be created; otherwise it will simply be opened.

Then we can open the connection, by calling the **OpenAsync()** method: it accepts, eventually, a parameter to set the opening mode (like read only). If we don’t set it, it will be used the default one, that supports read and write operations.

Then, we go manual! As already anticipated, the wrapper doesn’t support LINQ operations, so you’ll have to write the needed SQL statements to perform the operation. I assume that you already know the basics of SQL, so I won’t describe in details the queries: in this sample we define a query to create a table called **People** with 3 columns: **Id** (which is the primary key and it’s an auto increment value), **Name** and **Surname** (which simply contains strings).

To execute the query we call the async method **ExecuteStatementAsync()** on the **Database** object, passing as parameter the string with the query statement. **ExecuteStatementAsync()** is the method we need to use when the query doesn’t return any value and when we don’t need to define any parameter (we’ll see later how to use them).

As you can see, there&#8217;s a huge difference with the sqlite-net approach: there&#8217;s no mapping with a class and there&#8217;s no automatic conversion to a table. Everything is made with plain SQL statements.

### Perform operations on the database

The approach to perform operations (insert, update, select, etc.) is the same we’ve seen: we open the connection, we define the query statement and we execute the query. The only difference is that when you do an insert, for example, usually you need to set some parameters, because some elements of the query are not fixed but dynamic. For example, if we want to add a new row in the **People** table we’ve created in the step before, we need to pass two dynamic values: **Name** and **Surname**.

Here is the code to perform this operation:

<pre class="brush: csharp;">private async void OnAddDataClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "INSERT INTO PEOPLE (Name, Surname) VALUES (@name, @surname)";
    Statement statement = await database.PrepareStatementAsync(query);
    statement.BindTextParameterWithName("@name", "Matteo");
    statement.BindTextParameterWithName("@surname", "Pagani");

    await statement.StepAsync();

    statement = await database.PrepareStatementAsync(query);
    statement.BindTextParameterWithName("@name", "John");
    statement.BindTextParameterWithName("@surname", "Doe");

    await statement.StepAsync();
}</pre>

Please enter the **Statement** class, that can be used to perform additional operations on the query, like adding parameters and iterating over the results. To prepare a **Statement** object, you’ll need to call the **PrepareStatementAsync()** method passing as parameter the query to execute. How to manage parameters? The simplest way is to use **named parameters,** which can be added to a query simply by prefixing the @ symbol to the name of the parameter. In the sample, the insert query accepts two parameters: **@name** and **@surname**.

How to define the value of these parameters? By calling the **BindTextParameterWithName()** method on the **Statement** object with two parameters: the first one is the name of the parameter, the second one is the value we want to assign. In the sample, we’re going to add two users to the table: one with name **Matteo Pagani** and one with name **John Doe**. To execute the query, we call the method **StepAsync()** on the **Statement** object. There are also other versions of the **BindParameterWithName()** method, according to the data type of the parameter (for example, if it’s a number you can use the method **BindIntParameterWithName()**).

But the **StepAsync()** method has another purpose: it can be used also to iterate through the results of the query, in case we’re performing a query that can return one or more values. Let’s see, for example, how to perform a select to retrieve the data we’ve just inserted:

<pre class="brush: csharp;">private async void OnGetDataClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "SELECT * FROM PEOPLE";
    Statement statement = await database.PrepareStatementAsync(query);

    while (await statement.StepAsync())
    {
        MessageBox.Show(statement.GetTextAt(0) + " " + statement.GetTextAt(1));
    }
}</pre>

The first part of the code is the same we’ve seen before: we define the query (in this case, we retrieve all the rows of the **People** table), we prepare a **Statement** and we execute it with the **StepAsync()** method. The difference is that, this time, the **StepAsync()** method is performed into a **while** statement: it’s because the method will iterate over all the rows returned by the query so, every time we enter into the **while** loop, a new row is retrieved. In this sample, we expect to see the MessageBox twice: one for the user **Matteo Pagani** and one for the user **John Doe**. In the sample you see also how to get the values from the results: the **Statement** object offers some methods that starts with the **Get** prefix, that accepts as parameter the column index of the value to retrieve. There are some method for the most common data types: in this sample, since both **Name** and **Surname** are strings we use the **GetTextAt()** method, passing 0 as index to get the name and 1 as index to get the surname.

Of course we can combine what we’ve learned in the last two samples and, for example, we can perform a select query that contains some parameters:

<pre class="brush: csharp;">private async void OnGetSomeDataClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "SELECT * FROM PEOPLE WHERE Name=@name";
    Statement statement = await database.PrepareStatementAsync(query);
    statement.BindTextParameterWithName("@name", "Matteo");

    while (await statement.StepAsync())
    {
        MessageBox.Show(statement.GetTextAt(0) + " " + statement.GetTextAt(1));
    }
}</pre>

In this sample, we retrieve from the **People** table only the rows where the column **Name** contains the value **Matteo.**

We have also another option to access to the columns of the rows we’ve retrieved with a query, but it’s disabled by default because it slower and it returns every value as string, instead of its proper type. To enable it you have to call the **EnableColumnsProperty** method of the **Statement** object: once you’ve done it, you can access to the� values by using the **Columns** property of the **Statement** object: it’s a collection and you can access to each item by using the column’s name as index. Here is a sample:

<pre class="brush: csharp;">private async void OnGetSomeDataWithColumnsPropertyClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "SELECT * FROM PEOPLE";
    Statement statement = await database.PrepareStatementAsync(query);

    statement.EnableColumnsProperty();

    while (await statement.StepAsync())
    {
        MessageBox.Show(statement.Columns["Name"] + " " + statement.Columns["Surname"]);
    }
}</pre>

### Coming soon

In this post we’ve learned the basic concepts to use this new wrapper to perform operations on a SQLite database and which are the differences with another popular wrapper, sqlite-net. In the next post we’ll see how to deal with relationships, one of the most problematic scenarios to manage nowadays. Meanwhile, you can play with the sample project.

<div class="wlWriterEditableSmartContent" id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:0d1bf62b-b0ee-4386-aa38-04353bbf9dd9" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/06/SQLite.zip" target="_blank">Download the sample project</a>
  </p>
</div>