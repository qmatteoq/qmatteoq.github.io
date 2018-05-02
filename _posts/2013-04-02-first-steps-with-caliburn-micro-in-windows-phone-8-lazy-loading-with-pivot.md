---
id: 2702
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Lazy loading with Pivot'
date: 2013-04-02T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2702
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
After that, in the previous posts, we’ve talked about using the Panorama and Pivot controls in a Windows Phone application with Caliburn Micro, we’re ready to face the lazy loading topic. What is lazy loading? This approach is commonly used when working with databases and, simplifying, means that the data is loaded only when you really need it. Basically all the available ORM technologies (like Entity Framework or NHibernate) support lazy loading; think about a database, with many tables and relationships. Usually, when you do a query to retrieve the data that is the result of a join between two or more tables, the operation is immediately executed and all the requested data is retrieved from the database. With the lazy loading approach, instead, we are able to request for the data only when we need to work with it. Let’s say you have an application used to display orders and you want to do a query to get all the orders with the information about the customers that made them. With lazy loading, you are able to get all the orders and query for the related customer only when the user, actually, chooses to see his details: otherwise, the query is not executed.

How can this concept be applied to Pivot? With Caliburn Micro we’ve been able to split the different pages in the different Views and ViewModels but, in the end, they are all part of a single page: the conductor’s one, that contains the **Pivot** control. This means that, if we use the usual approach to load the data in the ViewModel’s constructor, when the user navigates to the page all the ViewModels are loaded and all the data is loaded and displayed.

Think about a news reader application, that displays different news categories in the different pages of a Pivot control: when the page is loaded, all the news are loaded, even the ones that are displayed in pages that are initially hidden. This is when lazy loading comes in handy, to improve the performances of our application: we can load just the data on the page that the user is currently viewing and loading the other stuff only when he swipes on that specific page.

Let’s see how to implement it: we’ll start from the same application we’ve developed in the last post, with a Pivot control and two pages.

**IMPORTANT!** Even if, from a code point of view, Panorama and Pivot have the same approach, we’ll be able to support lazy loading just using a Pivot, due to a bug in the Panorama control in Windows Phone 8. Which is the problem? That if Panorama items are added to the Panorama control using binding and the **ItemsSource** property (and they aren’t directly declared in the XAML or manually added in the code behind), the **SelectedIndex** property (which is important to keep track of the view that is actually visible) doesn’t work properly and returns only the values 0 and 1. The consequence is that the **SelectedItem** property doesn’t work properly, so we are not able to support lazy loading because we don’t know exactly when a view is displayed.

We’re going to develop a simple RSS reader, so first we’ll need some helpers class to accomplish our task.

The first one is the **FeedItem** class, which represents a single news from the RSS feed:

<pre class="brush: csharp;">public class FeedItem
{
    public string Title { get; set; }
    public string Description { get; set; }
}</pre>

It’s a simple class that is used to store the title and the description of the news. Then we need a simple parser, to convert the XML that we get into a collection of **FeedItem** objects:

<pre class="brush: csharp;">public static class RssParser
{
    public static IEnumerable&lt;FeedItem&gt; ParseXml(string content)
    {
        XDocument doc = XDocument.Parse(content);
        var result = doc.Descendants("rss").Descendants("channel").Elements("item").Select(x =&gt; new FeedItem
                                                                {
                                                                    Title = x.Element("title").Value,
                                                                    Description = x.Element("description").Value
                                                                });

        return result;
    }
}</pre>

By using LINQ to XML we are able to parse the RSS file and to extract, for every news (stored in the **item** node), the properties **title** and **description**.

Now we are ready to add the needed code to load the data: first let’s try to do it in the usual way, so we can see the difference. Our goal is to display a list news in the two Pivot pages, so we need, in the View, a ListBox to display them and, in the ViewModel, to retrieve the RSS, parse it and assign to a collection of **FeedItem** objects.� Here is the XAML:

<pre class="brush: xml;">&lt;ListBox x:Name="FeedItems"&gt;
    &lt;ListBox.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Path=Title}" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/ListBox.ItemTemplate&gt;
&lt;/ListBox&gt;</pre>

Nothing special to highlight: the **ItemTemplate** is really simple, we simply show the title of every news. We give to the **ListBox** control the name **FeedItems**: <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">if you remember the post about the collections naming convention</a>, we expect to have, in the ViewModel, a collection called with the same name, that will host the data to display. And here is the ViewModel:

<pre class="brush: csharp;">public class PivotItem1ViewModel: Screen
{
    public PivotItem1ViewModel()
    {
        DisplayName = "Pivot 1";
        LoadData();
    }

    public async void LoadData()
    {
        WebClient client = new WebClient();
        string result = await client.DownloadStringTaskAsync("http://feeds.feedburner.com/qmatteoq_eng");
        IEnumerable&lt;FeedItem&gt; feedItems = RssParser.ParseXml(result);

        FeedItems = feedItems.ToList();
    }

    private List&lt;FeedItem&gt; _feedItems;

    public List&lt;FeedItem&gt; FeedItems
    {
        get { return _feedItems; }
        set
        {
            _feedItems = value;
            NotifyOfPropertyChange(() =&gt; FeedItems);
        }
    } 
}</pre>

When the ViewModel is initialized we call the **LoadData()** method which, using the **WebClient** class and the <a href="http://wp.qmatteoq.com/async-targeting-pack-for-visual-studio-2012-why-its-useful-also-for-windows-phone-8-projects/" target="_blank">Async Targeting Pack</a> (so that we can use the **DownloadStringTaskAsync()** async method), we download the RSS feed. Then, we parse the result using the **RssParser** class we’ve created before and, in the end, we convert the collection we get in return in a **List<T>** object, that we assign to a property in the ViewModel called **FeedItems**. Due to the naming convention, this is the collection that will be displayed in the **ListBox** in the View.

Now repeat the same steps for the second page of the Pivot, by adding the same code to the **PivotItem2View** page and to the **PivotItem2ViewModel** class: just change the URL of the RSS feed to download with another one, so that we can see the differences.

Now run the application: you’ll see that, when the Pivot page is loaded, both views will load and display the list of news from the RSS item; not just the one that is currently displayed, but also the second one.

If you have many views in the Pivot and you need to load a lot of data, this operation can take some time and the user will have to wait that everything is fully loaded before using the application. So, let’s implement lazy loading! The operation is really simple and, if you have already read my previous posts about Caliburn Micro, you should already have a hint about the way to do it: using the navigation events.

<a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">In this post</a> we’ve talked about the **Screen** class and how, by inheriting our ViewModels from it, we are able to hook to many navigation events: specifically, the **OnActivate()** method is raised when the view is displayed. In a Panorama’s or Pivot’s context, this event is raised when the user swipes to that specific view: this means that we are able to use it to load the data only when the user navigates to that specific page. It’s enough to move the code inside the **LoadData()** method in the **OnActivate()** one to accomplish our goal:

<pre class="brush: csharp;">public class PivotItem1ViewModel: Screen
{
    public PivotItem1ViewModel()
    {
        DisplayName = "Pivot 1";
    }

    protected override async void OnActivate()
    {
        base.OnActivate();
        WebClient client = new WebClient();
        string result = await client.DownloadStringTaskAsync("http://feeds.feedburner.com/qmatteoq_eng");
        IEnumerable&lt;FeedItem&gt; feedItems = RssParser.ParseXml(result);

        FeedItems = feedItems.ToList();
    }

    private List&lt;FeedItem&gt; _feedItems;

    public List&lt;FeedItem&gt; FeedItems
    {
        get { return _feedItems; }
        set
        {
            _feedItems = value;
            NotifyOfPropertyChange(() =&gt; FeedItems);
        }
    } 
}</pre>

If you launch again the application your notice that, now, when the Pivot is loaded only the news in the first page will be displayed: the second page won’t contain any data. As soon as we swipe to the second page, the load operation is triggered and the list of news is displayed in the **ListBox**.

### That’s all

This is the conclusion of the series of posts about using the Pivot control in a Windows Phone application developed using the MVVM pattern with the help of Caliburn Micro.

<div class="wlWriterEditableSmartContent" id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:7fc3fba2-354d-4d66-8cfd-8f27a495c4c8" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    <b>The Caliburn Micro posts series</b>
  </p>
  
  <ol>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a>
    </li>
    <li>
      <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a>
    </li>
    <li>
      Lazy loading with pivot
    </li>
  </ol>
</div>