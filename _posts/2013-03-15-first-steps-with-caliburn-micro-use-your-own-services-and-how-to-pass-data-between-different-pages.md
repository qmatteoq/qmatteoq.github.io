---
id: 2162
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Use your own services'
date: 2013-03-15T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2162
permalink: /first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
In this series of posts about Caliburn Micro we’ve learned that the toolkit includes many built in helpers and services, that are automatically registered when, in the bootstrapper, you call the **RegisterPhoneServices()** method of the **PhoneContainer** class. We’ve seen many examples of these services, like **NavigationService** and **EventAggregator**. We’ve been able to get a reference to these services by simply adding a parameter in the view model’s constructor.

But what to do if we need to register your own classes and services? Let’s say that you&#8217;re developing a RSS reader application, that is able to parse the XML of a RSS feed and display it in a Windows Phone application. In this case, you may have a service (that is simply a dedicated class) that takes care of downloading the RSS and to convert it from a plain string to real objects. One approach would be to create an instance of the service directly in the ViewModel and use it, for example:

<pre class="brush: csharp;">public MainPageViewModel() 
{
    IFeedService feedService = new FeedService();
     // do something with the feed service
}</pre>

But this approach comes with a downside. Let’s say that you need to do some testing with fake data and, as a consequence, you want to switch the **FeedService** class with a fake feed service, that takes the data from a local XML file instead of using the real RSS feed. In a real scenario, probably your **FeedService** is used across multiple view models: in this case, you will have to go in every ViewModel and change the class that is used, like this:

<pre class="brush: csharp;">public MainPageViewModel() 
{
    IFeedService feedService = new FakeFeedService();
     // do something with the fake feed service
}</pre>

A good approach would be to use the same one that has been adopted by Caliburn Micro: services are registered at startup using the built in dependency container so that it’s enough to add a parameter in the ViewModel to have the service automatically resolved at runtime and injected into the ViewModel. This way, if we need to change the implementation of our service, we will do it just in the bootstrapper, when the services are registered: automatically every ViewModel will start to use it.

Here is how the ViewModel definition will change:

<pre class="brush: csharp;">public MainPageViewModel(IFeedService feedService)
{
    //do something with the feed service
}</pre>

### Register your services

Let’s see how to implement your own services and register them, so that you’ll able to use the approach I’ve explained. First you need to create an interface for your service, that describes the methods that the class will expose. This is an important step because it will allow you to easily switch the implementation of the class with another one just by changing the way it’s registered in the bootstrapper. Let’s take the previous example about the fake feed reader class: both the real and fake feed service will inherit from the **IFeedService** interface and, in the ViewModel constructor, we will ask for a reference to that interface. This way, when we change the implementation in the boostrapper, everything will continue to work fine.

Let’s see a sample of the interface for our **FeedService** class:

<pre class="brush: csharp;">public interface IFeedService
{
    Task&lt;List&lt;FeedItem&gt;&gt; GetNews(string url);
}</pre>

The interface describes just one method, that we will use to get the news list from the RSS feed: as parameter, it takes the URL of the RSS feed and returns a collection of **FeedItem** object, that is a simple class that describes some basic properties of a news item:

<pre class="brush: csharp;">public class FeedItem
{
    public string Title { get; set; }
    public string Description { get; set; }
    public Uri Url { get; set; }
}</pre>

And here is the real implementation of the **FeedService** class:

<pre class="brush: csharp;">public class FeedService: IFeedService
{
    public async Task&lt;List&lt;FeedItem&gt;&gt; GetNews(string url)
    {
        WebClient client = new WebClient();
        string content = await client.DownloadStringTaskAsync(url);

        XDocument doc = XDocument.Parse(content);
        var result = doc.Descendants("rss").Descendants("channel").Elements("item").Select(x =&gt; new FeedItem
        {
            Title = x.Element("title").Value,
            Description = x.Element("description").Value
        }).ToList();

        return result;
    }
}</pre>

In the class we actually write the real implementation of the **GetNews** method: it uses the **WebClient** class to download the RSS file (we use the **DownloadStringTaskAsync()** method that has been added by installing the <a href="http://wp.qmatteoq.com/async-targeting-pack-for-visual-studio-2012-why-its-useful-also-for-windows-phone-8-projects/" target="_blank">Async pack</a> using NuGet) and then, thanks to LINQ to XML, we extract the information we’re looking for (the title and the description of the news) and we store them in a new **FeedItem** object. At the end of the process, we have a collection of **FeedItem** objects:&nbsp; each one of them contains a news that was stored in the RSS file.

Now it’s time to register our service, so that it can be used by our ViewModel. The registration is made in the bootstrapper and&nbsp; you should be already familiar with it, since we’ve learned to register our view models every time we have added a new page to our project. We do it in the **Configure()** method of the bootstrapper class:

<pre class="brush: csharp;">protected override void Configure()
{
    container = new PhoneContainer(RootFrame);

    container.RegisterPhoneServices();
    container.PerRequest&lt;MainPageViewModel&gt;();
    container.PerRequest&lt;IFeedService, FeedService&gt;();
    AddCustomConventions();
}</pre>

We can see a difference: since our **FeedService** has an interface, we need to register it in a different way than we did for the **MainPageViewModel**. In fact, we have to tell to the container that, every time a ViewModel requires an **IFeedService** object, we want to provide a **FeedService** implementation. For this reason, the container exposes a **PerRequest<T, Y>** overload, where T is the base interface and Y is the real implementation of the interface.

Now we are able to just use it in the view model of our page, to display the list of news. Here is a sample XAML of the page:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;Button Content="Load website" x:Name="LoadWebsite"&gt;&lt;/Button&gt;
    &lt;ListBox x:Name="News"&gt;
        &lt;ListBox.ItemTemplate&gt;
            &lt;DataTemplate&gt;
                &lt;StackPanel&gt;
                    &lt;TextBlock Text="{Binding Path=Title}"&gt;&lt;/TextBlock&gt;
                    &lt;TextBlock Text="{Binding Path=Description}"&gt;&lt;/TextBlock&gt;
                &lt;/StackPanel&gt;
            &lt;/DataTemplate&gt;
        &lt;/ListBox.ItemTemplate&gt;
    &lt;/ListBox&gt;
&lt;/StackPanel&gt;</pre>

We have a button, that will execute the **LoadWebsite** method of the ViewModel (that will use our service to load the data), and we have a ListBox, which **ItemTemplate** simply displays the tile and the description of the news, one below the other.

And here is the ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel: Screen
{
    private readonly IFeedService feedService;

    private List&lt;FeedItem&gt; news;

    public List&lt;FeedItem&gt; News
    {
        get { return news; }
        set
        {
            news = value;
            NotifyOfPropertyChange(() =&gt; News);
        }
    }

    public MainPageViewModel(IFeedService feedService)
    {
        this.feedService = feedService;
    }

    public async void LoadWebsite()
    {
        News = await feedService.GetNews("http://feeds.feedburner.com/qmatteoq_eng");
    }
}</pre>

Nothing special here, except that we’ve added in the constructor a parameter which type is **IFeedService:** since we’ve registered it in the boostrapper, the parameter will contain a **FeedService** object, that we can use in the **LoadWebsite()** method to get the list of news using the **GetNews()** method we’ve defined in the service. This sample, we are parsing the RSS feed of this blog.

### Use your own service to pass data between pages

When <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">we talked about navigation</a> we learned that Caliburn exposes a way to pass data between pages, that relies on the query string parameter that are supported by the OS. The problem of this approach is that, since we’re talking about parameters that are added to the navigation url, we can only send text parameters, like strings and numbers. In most of the cases, instead, we need to pass complex object. Take as example the RSS reader app we’ve just built: we want to implement a detail page, that displays all the information about the news selected by the user. In this case, when we navigate to the detail page, we want to carry the whole **FeedItem** object that has been selected.

One approach is to use your own service to store the data and pass it to every ViewModel that needs the information, exactly like we did for the **FeedService**.&nbsp; Here is a sample of a **DataService** class:

<pre class="brush: xml;">public class DataService
{
    public FeedItem SelectedItem { get; set; }
}</pre>

As you can see it’s really simple, since it will be used just to store the **FeedItem** object that has been selected by the user in the ListBox.

Now we need to register it in the boostrapper:

<pre class="brush: csharp;">protected override void Configure()
{
    container = new PhoneContainer(RootFrame);

    container.RegisterPhoneServices();
    container.PerRequest&lt;MainPageViewModel&gt;();
    container.PerRequest&lt;DetailPageViewModel&gt;();
    container.PerRequest&lt;IFeedService, FeedService&gt;();
    container.Singleton&lt;DataService&gt;();
    AddCustomConventions();
}</pre>

And here comes something new: we’re not registering it using the familiar **PerRequest<T>** method, but with the **Singleton<T>** method exposed by the **PhoneContainer** class. Which is the difference? When a class is registered using the **PerRequest<T>** method every time a ViewModel asks for a reference, it gets a new instance of the object. It’s the best approach when we’re dealing with view models: think about the **DetailPageViewModel** we’ve registered, that is connected to the page that displays the details of the selected news. In this case, every detail page will be different, because we’ll have to display a different news: by creating a new instance of the view model every time the user navigates to the detail page we make sure that the fresh data is correctly loaded.

This is not the case for our **DataService:** in this case we need to maintain a single instance across the application, because we want to take back and forth the **FeedItem** object selected by the user. If we would have registered it using the **PerRequest<T>** method, we would have lost the value stored in the **SelectedItem** property as soon as the user navigates away from the main page. The answer is using the **Singleton<T>** method: this way we’ll always get the same object in return when a ViewModel asks for it.

Now we just need to add a parameter which type is **DataService** in the constructor of both our view models: the main page one and the detail page one.

<pre class="brush: csharp;">public class MainPageViewModel: Screen
{
    private readonly IFeedService feedService;
    private readonly INavigationService navigationService;
    private readonly DataService dataService;

    private List&lt;FeedItem&gt; news;

    public List&lt;FeedItem&gt; News
    {
        get { return news; }
        set
        {
            news = value;
            NotifyOfPropertyChange(() =&gt; News);
        }
    }

    private FeedItem selectedNew;

    public FeedItem SelectedNew
    {
        get { return selectedNew; }
        set
        {
            selectedNew = value;
            dataService.SelectedItem = value;
            navigationService.UriFor&lt;DetailPageViewModel&gt;().Navigate();
            NotifyOfPropertyChange(() =&gt; SelectedNew);
        }
    }

    public MainPageViewModel(IFeedService feedService, INavigationService navigationService, DataService dataService)
    {
        this.feedService = feedService;
        this.navigationService = navigationService;
        this.dataService = dataService;
    }

    public async void LoadWebsite()
    {
        News = await feedService.GetNews("http://feeds.feedburner.com/qmatteoq_eng");
    }
}</pre>

We’ve added a property called **SelectedNew**: if you remember what I’ve explained <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">in this post</a>, you’ll know that by using this naming convention we are able to get automatically, in the **SelectedNew** property, the item selected by the user in the **ListBox** that is connected to the **News** collection.

Inside the **setter** of this property we do two additional things: the first one is to store the selected value in the **SelectedItem** property of the **DataService** class. The second is to redirect the user to the detail page, using the built in **NavigationService**.

What about the second page? The view it’s very simple, since it’s used just to display the **Title** and **Description** properties of the selected item.

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;TextBlock x:Name="Title" /&gt;
    &lt;TextBlock x:Name="Description" /&gt;
&lt;/StackPanel&gt;</pre>

And the ViewModel is really simple too:

<pre class="brush: csharp;">public class DetailPageViewModel: Screen
{
    private string title;

    public string Title
    {
        get { return title; }
        set
        {
            title = value;
            NotifyOfPropertyChange(() =&gt; Title);
        }
    }

    private string description;

    public string Description
    {
        get { return description; }
        set
        {
            description = value;
            NotifyOfPropertyChange(() =&gt; Description);
        }
    }

    public DetailPageViewModel(DataService dataService)
    {
        Title = dataService.SelectedItem.Title;
        Description = dataService.SelectedItem.Description;
    }
}</pre>

We add a parameter in the constructor which type is **DataService**: this way the bootstrapper will give us the correct instance that, since has been registered as singleton, will be the same that was used by the **MainPageViewModel**. This way, the **SelectedItem** property of the **DataService** will contain the item selected by the user in the main page: we simply use it to set the **Title** and **Description** properties, so that they are displayed in the page.

That’s all for this topic! But don’t be afraid, there are some other subjects to talk about in the next posts <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Smile" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/wlEmoticon-smile1.png?w=640" data-recalc-dims="1" />

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:32439df1-c430-4e69-8402-cacf9f65ee8a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_Services.zip" target="_blank">Download the sample project</a>
  </p>
</div>

&nbsp;

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a> 
          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a> 
              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a> 
                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a> 
                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a> 
                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                  * Use your own services 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>