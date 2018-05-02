---
id: 238
title: 'Having fun with Azure Mobile Services &#8211; Integrating with Windows 8'
date: 2012-10-04T10:00:06+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=238
permalink: /having-fun-with-azure-mobile-services-integrating-with-windows-8/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Azure
  - Windows 8
  - Windows Phone
---
In the previous post we’ve introduced Azure Mobile Services and we learned how to configure and create them. If you’ve followed all the steps of the previous post, you should have now a service up & running that allows to interact with a table called **Comics**, that we’ve created to store information about our favourite comics.

In this post we’ll see how to interact with this service from a Windows 8 application: as I’ve anticipated in the previous post, Windows 8 is the easiest platform to integrate with our service, since Microsoft has released a specific SDK for Windows Store apps. This SDK basically hides all the web requests that, under the hood, are exchanged with the service and automatically serialize and deserialize our data.

The first thing, indeed, is to download the SDK <a href="http://go.microsoft.com/fwlink/?LinkId=257545&clcid=0x409" target="_blank">from the following link.</a> After you’ve installed it, it’s time to open Visual Studio 2012 and create a new Windows Store app (you can use the blank template, in this post we’ll simply learn how to communicate with our service, we won’t develop a real application with full graphic).

After installing the SDK you’ll find a library in the **Windows – Extensions** section of the **Add new reference** dialog: it’s called **Windows Azure Mobile Services Managed Client** and you simply have to double click on it to add it to your project.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb7.png?resize=496%2C186" alt="image" width="496" height="186" border="0" data-recalc-dims="1" />](https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image7.png)

&nbsp;

As I already anticipated, the app will be very simple: no graphic, no user experience, just two buttons, one to store data and one to retrieve it and display it in a ListView. So let’s start by adding them in the XAML of our MainPage:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;Button Content="Insert data" Click="OnAddNewComicButtonClicked" /&gt;
    &lt;Button Content="Show data" Click="OnGetItemButtonClicked" /&gt;
    &lt;ListView&gt;
        &lt;ListView.ItemTemplate&gt;
            &lt;DataTemplate&gt;
                &lt;StackPanel Margin="0, 20, 0, 0"&gt;
                    &lt;TextBlock Text="{Binding Title}" /&gt;
                    &lt;TextBlock Text="{Binding Author}" /&gt;
                &lt;/StackPanel&gt;
            &lt;/DataTemplate&gt;
        &lt;/ListView.ItemTemplate&gt;
    &lt;/ListView&gt;
&lt;/StackPanel&gt;</pre>

The code should be simple to understand: with the first button we’re going to store some data in our service; with the second one we’re going to retrieve it and displaying it in the below **ListView**, which simply shows, one below the other, the title and the author of the comic.

### 

### Let’s prepare the application

Before starting to do some operation we’ll need to create the class that maps the data we have on our service: since we’ve created a **Comic** table, we’re going to create a **Comic** class with, as properties, the columns of our table. Here is the code:

<pre class="brush: csharp;">public class Comic
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}</pre>

The second thing to is to initialize the client we’re going to use to do operations with the service: we can do it, for example, in the constructor of our MainPage, by declaring it at class level (so that every method we’re going to write will be able to use it). Initializing the client is very simple:

<pre class="brush: csharp;">MobileServiceClient client = new MobileServiceClient("https://myService.azure-mobile.net/",
"your-application-key");</pre>

You’re going to use the class **MobileServiceClient** (it’s inside the **Microsoft.WindowsAzure.MobileServices** namespace) that, when it’s initalized, requires two parameters: the first one is the address of your service (the one we have chosen when we have configured our service), the second one is the secret application key. To get your key, simply open the Azure Management Portal and, in the home page of your Azure Mobile Service, choose the option **Connect to an exisiting Windows Store app**. In the window that will appear you will find, at step 2, the same code I’ve just written, but already filled with the correct data of your service. Just copy and paste it in your application and you’re done!

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb8.png?resize=585%2C183" alt="image" width="585" height="183" border="0" data-recalc-dims="1" />](https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image8.png)

### 

### Insert some data

Now that we have the client available, we can start to see how to insert some data in our servie, by managing the event handler of the first button we’ve defined in the XAML. Here is the code:

<pre class="brush: csharp;">private async void OnAddNewComicButtonClicked(object sender, RoutedEventArgs e)
{
    MobileServiceClient client = new MobileServiceClient("https://myService.azure-mobile.net/",
                                                        "your-application-key");

    Comic comic = new Comic
                       {
                           Title = "Batman Year One",
                           Author = "Frank Miller",
                       };

    await client.GetTable&lt;Comic&gt;().InsertAsync(comic);

}</pre>

First we create a new **Comic** object, with a title and an author. Then, using the **MobileServiceClient** object, we get a reference to the **Comic** table and, in the end, we call the **InsertAsync** method by passing as parameter the comic object we’ve just created. This operation is awaitable (it can require some time to be executed, since it involves communications with a service over Internet), so we’re going to use the magic keywords async (in the event handler’s signature) and await (before the method, in order to await that the operation is ended before moving on).

If you go back to the Azure Management Portal and you access to your service’s dashboard, in the **Data** section you’ll find that the new item has just been added.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb9.png?resize=532%2C115" alt="image" width="532" height="115" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image9.png)

### Using dynamic data to add columns to the table

If you remember what we did in the previous post, after we created our **Comics** table we’ve added some columns using the Azure management tool. At the same time, I also told you that this step wasn’t really necessary, thanks to a feature called **dynamic data,** that is able to add new columns to the table by simply adding an item that contains new properties other than the ones already stored.

Let’s see how to use it: first add a new property in your **Comic** class called **Publisher;** we’re going to use it to store the publisher’s name of the comic.

<pre class="brush: csharp;">public class Comic
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public string Publisher { get; set; }
}</pre>

Now let’s edit the button’s event handler to add a new **Comic** object: this time we’ll set also the **Publisher**’s property of the project before inserting it.

<pre class="brush: csharp;">private async void OnAddNewComicButtonClicked(object sender, RoutedEventArgs e)
{
    MobileServiceClient client = new MobileServiceClient("https://myService.azure-mobile.net/",
                                                        "your-application-key");

    Comic comic = new Comic
                       {
                           Title = "Watchmen",
                           Author = "Alan Moore",
                           Publisher = "DC Comics"
                       };

    await client.GetTable&lt;Comic&gt;().InsertAsync(comic);
}</pre>

Run this code and you’ll see that, despite the fact that you’re adding a **Comic** object with a property that is missing in the table, you won’t get any exception. Go back to the Azure dashboard and, in the **Data** section, you’ll find that a new column called **Publisher** has been added: obviously, you’ll find a value only for the item you’ve just added, while the previous one will have an empty value.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb6.png?resize=532%2C144" alt="image" width="532" height="144" border="0" data-recalc-dims="1" />](https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image6.png)

According to what we’ve learned about the **dynamic data** we could have avoided, in the last post, to create the columns using the Azure management tool: we could have simply inserted a **Comic** object with the **Title** and **Author** properties and the service would have done everything for us.

### 

### How to work with the data

At this time it shouldn’t be too hard to understand how to get the data we stored on our service: by using the **GetTable<>()** method we’ve just seen we get a reference to the table. This table object (which type is **IMobileServiceTable<T>**) allows to perform operations using LINQ, so that we can filter the data before actually making the request. To get the real data we can use one of the available methods: **ToListAsync()** or **ToEnumerableAsync()**, that returns a collection of the objects stored in the table.

Here are some examples of the operations you can do:

<pre class="brush: csharp;">private async void OnGetItemButtonClicked(object sender, RoutedEventArgs e)
{
    MobileServiceClient client = new MobileServiceClient("https://myService.azure-mobile.net/",
                                            "your-application-key");

    //get all the comics
    List&lt;Comic&gt; allComics = await client.GetTable&lt;Comic&gt;().ToListAsync();

    //get all the comics which publisher is DC Comics
    List&lt;Comic&gt; filteredComics = await client.GetTable&lt;Comic&gt;().Where(x =&gt; x.Publisher == "DC Comics").ToListAsync();

    //get all the comics ordered by title
    List&lt;Comic&gt; orderedComics = await client.GetTable&lt;Comic&gt;().OrderBy(x =&gt; x.Title).ToListAsync();

    ComicsList.ItemsSource = allComics;
}</pre>

In these three examples you can see how to retrieve all the data, how to retrieve filtered data (all the comics with a specific publisher) and how to apply an order criteria to the results.

And if we want to manipulate the data already stored in the table? We can simply use:

  * the **UpdateAsync** method to update an item. We simply have to pass to the method the update object and, using the unique identifier (in our example, the **Id** property of the **Comic** class) the already existing item will be overwritten by the new one.
  * the **DeleteAsync** method to delete an item. In this case, we simply have to pass to the method the object to delete: the method will take care to find it in the table and to remove it.

### We’ve reached our goal… for the moment

In this post we’ve seen how to integrate our new Azure Mobile Service with a Windows 8 application. In the next post we’ll do the same with a Windows Phone application: things won’t be so easy as we’ve seen in this post, due to the temporary lack of a SDK for Windows Phone, but don’t worry, we’ll have fun anyway <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/wlEmoticon-smile.png?w=640" alt="Smile" data-recalc-dims="1" />