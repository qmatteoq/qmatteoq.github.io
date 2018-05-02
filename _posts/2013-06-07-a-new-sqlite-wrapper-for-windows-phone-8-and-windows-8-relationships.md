---
id: 3522
title: 'A new SQLite wrapper for Windows Phone 8 and Windows 8 &ndash; Relationships'
date: 2013-06-07T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3522
permalink: /a-new-sqlite-wrapper-for-windows-phone-8-and-windows-8-relationships/
categories:
  - Windows 8
  - Windows Phone
tags:
  - SQLite
  - Windows 8
  - Windows Phone
---
This post is the second part of a series about a new SQLite wrapper for Windows Phone 8 and Windows 8, <a href="http://wp.qmatteoq.com/a-new-sqlite-wrapper-for-windows-phone-8-and-windows-8-the-basics" target="_blank">you can read the first part here</a>.

One of the biggest flaws of the sqlite-net library for Windows 8 and Windows Phone 8 is that it doesn’t support relationships: there isn’t an attribute that can be used to mark a class property as a foreign key. Plus, sqlite-net offers some methods to manually execute a SQL statement, but the problem is that doesn’t offer a way to iterate through the results, but it always expects a specific object in return.

Let’s take a very common situation as example: to the database with the **People**� table we’ve created in the previous post we want to add a new table called **Orders**, to keep track of the orders made by the users. With sqlite-net we would have created two classes, one to map the **People** table and one to map the **Orders** table. Then, we could have execute a manual query in a similar way:

<pre class="brush: csharp;">private async void OnExecuteJoin(object sender, RoutedEventArgs e)
{
   SQLiteAsyncConnection conn = new SQLiteAsyncConnection(Path.Combine(ApplicationData.Current.LocalFolder.Path, "people.db"), true);
   string query = "SELECT * FROM People INNER JOIN Orders ON Orders.PersonId = People.Id";
   List&lt;Person&gt; personOrders = await conn.QueryAsync&lt;Person&gt;(query);
}</pre>

Which is the problem with the sqlite-net approach? That, as you can see, when you call the **QueryAsync()** method it requires a **<T>** parameter, which is the type that you expect as a result of the query. Sqlite-net will automatically deserialize the result and will provide the object for you, ready to be used in your application. Smart, isn’t it? The problem is that this approach doesn’t work when we have a relationship: the sample code that you see is wrong, because when we have created a relationship between the tables **People** and **Orders** we don’t expect anymore to get, as a result, a **Person** object or an **Order** object, but a combination of both of them. The workaround, in this case, would be to create a third class, something like **PeopleOrders**, that will contain all the properties of the **People** class combined with the properties of the **Orders** class. Suddenly, it doesn’t sound so smart, isn’t it? <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" alt="Smile" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />

With this new wrapper, instead, we’ll be able to support this scenario, because we can simply iterate trough the rows returned by the query and pick the data we need using the **Statement** object. The only downside is that we’ll have to do a lot of manual work: we’ll have to create our objects from scratch, since there is no automatic serialization and deserialization of the data.

Let’s see, step by step, how to support relationships with the new wrapper. We’ll reuse the knowledge <a href="http://wp.qmatteoq.com/a-new-sqlite-wrapper-for-windows-phone-8-and-windows-8-the-basics" target="_blank">we’ve learned in the previous post</a>.

### Let’s update the database

The first thing to do is to create a new table, that will store the new **Orders** table. Let’s see the code:

<pre class="brush: csharp;">private async void OnCreateDatabaseClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "CREATE TABLE PEOPLE " +
                   "(Id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL," +
                   "Name varchar(100), " +
                   "Surname varchar(100))";

    await database.ExecuteStatementAsync(query);
        string query2 = "CREATE TABLE ORDERS " +
                        "(Id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL," +
                        "Description VARCHAR(200)," +
                        "Amount INTEGER," +
                        "PersonId INTEGER, " +
                        "FOREIGN KEY(PersonId) REFERENCES People(Id) ON DELETE CASCADE)";

        await database.ExecuteStatementAsync(query2);
}</pre>

The **People** table is the same we’ve seen <a href="http://wp.qmatteoq.com/a-new-sqlite-wrapper-for-windows-phone-8-and-windows-8-the-basics" target="_blank">in the previous post</a>; the **Orders** table contains some columns to store the info about the order (description, amount, etc.) plus a column to manage the relationship, that will act as a foreign key. Specifically, we add a column called **PersonId**, that will store the id of the user that made the order taken from the **Id** column of the **People** table. We also define that this column is a foreign key and that, if we delete a user, all his orders will be deleted too (with the **ON DELETE CASCADE** statement). To define the key we use the following statement:

**FOREIGN KEY(PersonId) REFERENCES People(Id)**

that means that the **PersonId** column of the **Orders** table will hold a reference to the **Id** column of the **People** table.

### Manage the orders

Now we’re ready to start using the relationships and add an order made by the user **Matteo Pagani**, which **Id** (that has been auto generated) is 1 (we’ve added this user using the code from the previous post).

<pre class="brush: csharp;">private async void OnInsertOrderClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "INSERT INTO ORDERS(Description, Amount, PersonId) VALUES (@Description, @Amount, @PersonId)";
    Statement statement = await database.PrepareStatementAsync(query);
    statement.BindTextParameterWithName("@Description", "First order");
    statement.BindIntParameterWithName("@Amount", 200);
    statement.BindIntParameterWithName("@PersonId", 1);

    await statement.StepAsync();
}</pre>

If you’ve read my previous post, there should be nothing special in this code: we execute a insert query and we add, as parameters, the description of the order, the amount of the order and the id of the person that made the order. In the end, we execute the query using the **StepAsync()** method of the **Statement** object.

Now, it’s time to retrieve the data: let’s do a join statement, to retrieve all the orders with the information about the user that made it, like we’ve seen in the first sample with sqlite-net (the one that wasn’t working <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" alt="Smile" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />).

<pre class="brush: csharp;">private async void GetAllOrdersClicked(object sender, RoutedEventArgs e)
{
    Database database = new Database(ApplicationData.Current.LocalFolder, "people.db");

    await database.OpenAsync();

    string query = "SELECT * FROM ORDERS INNER JOIN PEOPLE ON ORDERS.PersonId=PEOPLE.Id";

    Statement statement = await database.PrepareStatementAsync(query);
    statement.EnableColumnsProperty();

    while (await statement.StepAsync())
    {
        MessageBox.Show(string.Format("{0} by {1} {2}", statement.Columns["Description"],
                                      statement.Columns["Name"], statement.Columns["Surname"]));
    }

}</pre>

The query is exactly the same: what changes is that, now, we’re able to iterate through the results so, thanks to the **Statement** object and the� **StepAsync()** method**,** we’re able to extract all the values we need (Description, Name and Surname) and display it using a MessageBox. In a real application, probably, we would have populated a collection of data, that we would have displayed in the application using a ListBox or a LongListSelector, for example. Just notice that I’ve enabled the columns property feature (with the **EnableColumnsProperty()** method), so that we can access to the columns directly using their names as index of the **Columns** collection.

### In the end

With the latest two posts we’ve seen a different approach to use SQLite in a Windows Phone 8 or Windows 8 app: with sqlite-net, we have an approach similar to the one we’ve learned with SQL CE in Windows Phone. Thanks to the power of LINQ, performing operations on database is really simple and, as with every other ORM, you can keep thinking using the objects approach, instead of having to deal with SQL. But all this flexibility comes with a price: the biggest limitation, right now, with sqlite-net, is that almost impossible to manage relationships. On the other side, with the new wrapper by Microsoft, you have full control, thanks to the great support to manual SQL statements; in complex applications, where you have to deal with lot of tables and relationships, it’s a big pro and it may help you a lot also to fine tuning performances; on the other side, you’ll have to deal on manual queries even for the most simple ones, like creating a table or adding some data. It’s up to you (and to the project you’re working on) to choose which wrapper is the best for you!

<div class="wlWriterEditableSmartContent" id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:4502d060-1b9e-4630-bccb-f528332e6716" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/06/SQLiteRelationships1.zip" target="_blank">Download the sample project</a>
  </p>
</div>