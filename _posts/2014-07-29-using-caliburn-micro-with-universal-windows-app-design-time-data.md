---
id: 6296
title: 'Using Caliburn Micro with Universal Windows app &ndash; Design time data'
date: 2014-07-29T15:30:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6296
permalink: /using-caliburn-micro-with-universal-windows-app-design-time-data/
categories:
  - Universal Apps
  - Windows 8
  - Windows Phone
tags:
  - Caliburn
  - Windows 8
  - Windows Phone
---
One of the most powerful features in XAML is design time data support. Let’s say that you have an application that displays some news retrieved from a RSS feed using a ListView or a GridView control. If you try to edit the visual layout of your application using Blend or the Visual Studio designer, you won’t be able to get a preview of the final result: since data is retrieved from a remote service, the ListView will look empty, since the designer isn’t able to load the real data. This scenario makes hard for a designer to have a real preview of the final result or to edit some elements (like the ItemTemplate of the ListView).

Design data is the solution to this problem: it’s basically a way to create fake data, that is loaded only when the View is displayed in Blend or in the Visual Studio designer. Let’s see how to manage this scenario using Caliburn Micro.

### The sample app: a simple news reader

As a sample, we’re going to develop a simple news reader, that will read and parse the RSS feed of this blog. The real application will display the real posts from the blog, while the designer will display some fake posts, just to give to designer an idea of the content that will displayed in the page. First, let’s start to set up the real application, using the conventions we’ve learned in the previous posts. The application will have just one View, that will display a list of posts using a ListView control.

First, let’s define a simple class that will map some of the information about the post that we want to display in the page:

<pre class="brush: csharp;">public class FeedItem
{
    public string Title { get; set; }
    public DateTimeOffset PublishDate { get; set; }
    public string Description { get; set; }
}
</pre>

Then, we need to download the RSS feed from the web and convert the XML into a list of **FeedItem** objects. For this purpose, we’re going to create a separate class, that will take care of processing the RSS feed: we’re going to define a method that will take, as input, the URL of the RSS feed and will return, as output, a collection of **FeedItem** objects. Here is how the interface of the class looks like:

<pre class="brush: csharp;">public interface IFeedService
{
    Task&lt;IEnumerable&lt;FeedItem&gt;&gt; GetNews(string feedUrl);
}</pre>

To implement it, we’re going to use one of the new 8.1 APIs, called **SyndicationClient:** it’s a special **HttpClient** implementation that is able to download the RSS feed and to automatically parse it, so that you can work with classes and objects instead of raw XML. Here is the implementation:

<pre class="brush: csharp;">public class FeedService: IFeedService
{
    public async Task&lt;IEnumerable&lt;FeedItem&gt;&gt; GetNews(string feedUrl)
    {
        SyndicationClient client = new SyndicationClient();
        SyndicationFeed feed = await client.RetrieveFeedAsync(new Uri(feedUrl, UriKind.Absolute));
        List&lt;FeedItem&gt; feedItems = feed.Items.Select(x =&gt; new FeedItem
        {
            Title = x.Title.Text,
            Description = x.Summary.Text,
            PublishDate = x.PublishedDate
        }).ToList();

        return feedItems;
    }
}
</pre>

We use the **RetrieveFeedAsync()** method, which takes in input the URL of the RSS feed (which is passed as parameter of the **GetNews()** method), and it will give you in return a **SyndicationFeed** object, which stores all the information about the feed. What’s interesting for us is the **Items** collection, which contains all the news that are included in the feed (in our case, the list of all the posts published in this blog). Using LINQ, we convert it into a collection of **FeedItem** objects, by grabbing the title, the description and the published date of the post.

Now we need to use this class in the ViewModel that is connected to our page. The easiest way would be to simply create a new instance of the **FeedService** class and to call the the **GetNews()** method, like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel : Screen
{

    public MainPageViewModel()
    {
    }

    private List&lt;FeedItem&gt; _news;

    public List&lt;FeedItem&gt; News
    {
        get { return _news; }
        set
        {
            _news = value;
            NotifyOfPropertyChange(() =&gt; News);
        }
    }

    protected override async void OnActivate()
    {
        IFeedService feedService = new FeedService();
        IEnumerable&lt;FeedItem&gt; items = await feedService.GetNews("http://feeds.feedburner.com/qmatteoq_eng");
        News = items.ToList();
    }
}
</pre>

However, this approach has some cons. Let’s say that, due to some changes in the requirements, you need to swap the **FeedService()** class with another implementation, by keeping ****the **IFeedService** interface as a base ground. With this approach, you would have to go in every ViewModel in which you’re using the **FeedService** class and replace the implementation. But wait, there’s a better way to do that: using the dependency injection container provided by Caliburn Micro!

Let’s move to the **App.xaml.cs** file and, in the **Configure()** method, let’s register the **FeedService**, like we did for the different ViewModels:

<pre class="brush: csharp;">protected override void Configure()
{
    container = new WinRTContainer();

    container.RegisterWinRTServices();

    container.PerRequest&lt;MainPageViewModel&gt;();
    container.PerRequest&lt;IFeedService, FeedService&gt;();
}
</pre>

The only difference is that, this time, since we have an interface that describes our class, we use another approach: instead of registering just the class (by calling the **PerRequest<T>()** method), we bind the interface with the concrete implementation, by using the **PerRequest<T,Y>()** method. Now, to get a reference to the **FeedService** class in our ViewModel, we simply need to add a new parameter in the constructor, which type is **IFeedService**, ****like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel : Screen
{
    private readonly IFeedService _feedService;

    public MainPageViewModel(IFeedService feedService)
    {
        _feedService = feedService;
    }

    private List&lt;FeedItem&gt; _news;

    public List&lt;FeedItem&gt; News
    {
        get { return _news; }
        set
        {
            _news = value;
            NotifyOfPropertyChange(() =&gt; News);
        }
    }

    protected override async void OnActivate()
    {
        IEnumerable&lt;FeedItem&gt; items = await _feedService.GetNews("http://feeds.feedburner.com/qmatteoq_eng");
        News = items.ToList();
    }
}
</pre>

We’ve simply added a parameter in the constructor which type is **IFeedService**: the dependency injection container will take care of resolving the parameter at runtime, by injecting the real implementation we’ve defined in the **App** class (in our case, **FeedService**). This way, if we need to use a different implementation of the **IFeedService** class, we simpy have to go back to the **App** class and change the **Configure()** method we’ve previously defined, instead of having to change the code in every ViewModel that needs to use this class. Now we can simply define our View using a ListView control, to display the posts retireved by the **FeedService**:

<pre class="brush: xml;">&lt;ListView ItemsSource="{Binding Path=News}"&gt;
    &lt;ListView.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Path=Title}" Style="{StaticResource TitleTextBlockStyle}" /&gt;
                &lt;TextBlock Text="{Binding Path=PublishDate}" Style="{StaticResource BodyTextBlockStyle}"&gt;&lt;/TextBlock&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/ListView.ItemTemplate&gt;
&lt;/ListView&gt;
</pre>

Now if you run the app (it doesn’t matter if the Windows 8.1 or the Windows Phone 8.1 version) you should see the lists of posts recently published on this blog, downloaded from the following RSS feed: [http://feeds.feedburner.com/qmatteoq_eng](http://feeds.feedburner.com/qmatteoq_eng "http://feeds.feedburner.com/qmatteoq_eng")

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/07/image_thumb.png?resize=277%2C504" width="277" height="504"  data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/07/image.png)

### Using design data

If you try to open your project in Blend or you enable the Visual Studio designer, you’ll notice that the page will be empty: the real **FeedService** isn’t executed during design time, so you don’t get a chance to see the list of posts. The purpose of our work will be to display a list of fake posts, in case the View is displayed inside the designer.

One thing you should have noticed, while working with Caliburn, is that you don’t get Intellisense while you’re dealing with binding, unlike when you work with other toolkits like MVVM Lights. This happens because you’re not explicitly setting the **DataContext** of the page: it’s automatically resolved at runtime, by using the specfic Caliburn naming convention. The result is that, when you’re writing the XAML, the View doesn’t know which is the ViewModel that is connected to it. This is a consequence of the design data requirement: we need to tell to the View which is the connected ViewModel, so that it can enable Intellisense and that it can provide design data for us.

Here is how to do it:

<pre class="brush: xml;">&lt;Page
    x:Class="DesignData.Views.MainPageView"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:DesignData"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:micro="using:Caliburn.Micro"
    xmlns:viewModels="using:DesignData.ViewModels"
    mc:Ignorable="d"
    d:DataContext="{d:DesignInstance Type=viewModels:MainPageViewModel, IsDesignTimeCreatable=True}"
    micro:Bind.AtDesignTime="True"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

    &lt;Grid&gt;
        &lt;ListView ItemsSource="{Binding Path=News}"&gt;
            &lt;ListView.ItemTemplate&gt;
                &lt;DataTemplate&gt;
                    &lt;StackPanel&gt;
                        &lt;TextBlock Text="{Binding Path=Title}" Style="{StaticResource TitleTextBlockStyle}" /&gt;
                        &lt;TextBlock Text="{Binding Path=PublishDate}" Style="{StaticResource BodyTextBlockStyle}"&gt;&lt;/TextBlock&gt;
                    &lt;/StackPanel&gt;
                &lt;/DataTemplate&gt;
            &lt;/ListView.ItemTemplate&gt;
        &lt;/ListView&gt;
    &lt;/Grid&gt;
&lt;/Page&gt;
</pre>

First we’ve defined which is the **DataContext** of the page, but we did it in a different way, by adding the prefix **d**: ****this way, we’re telling to the XAML compiler that this value should be used only at design time, while at runtime the application will continue to use the standard **DataContext** (in our case, the ViewModel assigned by Caliburn). With this definition, basically we’re manually defining what Caliburn automatically does at runtime: we’re telling to the current View that the **DataContext** class to use is called **MainPageViewModel** and that supports design time data.

Then, we need to use a special Caliburn attached property, that is able to perform the binding for us at design timed: it’s called **Bind.AtDesignTime** and its value has to be set to **True**. Please note that, to get this code working, we need to define two XAML namespaces: the first one (called **viewModels** in the sample) which points to the project’s folder that contains all our ViewModels (in our case, **using:DesignData.ViewModels**, where **DesignData** is the name of the project); the second one (called **micro** in the sample) it’s the default Caliburn one, which is **using:Caliburn.Micro**.

After this setup, one thing you’ll immediately notice is that you’ll get Intellisense up and running: if you try to setup a new binding with a control, Visual Studio will propose you one of the properties that have been declared in the ViewModel.

Now it’s time to define the design time data: one important thing to highlight is that this feature, to work properly in Caliburn, requires a parameterless constructor. In our case, it will be different than the standard one (which, instead, contains a parameter which type is **IFeedService**): in this empty constructor we’re going to set up the fake data, by filling the **News** collection (the one displayed in the **ListView** control) with a list of fake posts, like in the following sample.

<pre class="brush: csharp;">public class MainPageViewModel : Screen
{
    private readonly IFeedService _feedService;

    public MainPageViewModel(IFeedService feedService)
    {
        _feedService = feedService;
    }

    public MainPageViewModel()
    {
        if (Execute.InDesignMode)
        {
            News = new List&lt;FeedItem&gt;
            {
                new FeedItem
                {
                    Title = "First news",
                    Description = "First news",
                    PublishDate = new DateTimeOffset(DateTime.Now)
                },
                new FeedItem
                {
                    Title = "Second news",
                    Description = "Second news",
                    PublishDate = new DateTimeOffset(DateTime.Now)
                },
                new FeedItem
                {
                    Title = "Third news",
                    Description = "Third news",
                    PublishDate = new DateTimeOffset(DateTime.Now)
                }
            };
        }
    }

    private List&lt;FeedItem&gt; _news;

    public List&lt;FeedItem&gt; News
    {
        get { return _news; }
        set
        {
            _news = value;
            NotifyOfPropertyChange(() =&gt; News);
        }
    }

    protected override async void OnActivate()
    {
        IEnumerable&lt;FeedItem&gt; items = await _feedService.GetNews("http://feeds.feedburner.com/qmatteoq_eng");
        News = items.ToList();
    }
}
</pre>

As you can see, the main code of the ViewModel is the same: we have one constructor, that takes care of initializing the **IFeedService** object; we have a property called **News**, which contains a list of **FeedItem** objects; in the **OnActivate()** method (that is invoked when the view is displayed) we use the **FeedService** class to retrieve the list of posts and to display it in the View.

The difference is that now we have a constructor without parameters, in which we initialize a list of fake posts: we manually create some **FeedItem** objects and we add them to the **News** collection. Of course, we want to do that only if the View is displayed in the designer: we don’t want to display fake data when the app is running on the phone. For this purpose, we can use a Caliburn class called **Execute**, which provides some useful properties and methods: for our scenario we can use a property called **InDesignMode**, which is a simple boolean that is set to **true** in case the View is displayed inside the designer. We’re goint to add fake data to the **News** collection only if the value of this property is set to **true**.

Now, build your project and try to open the designer: you’ll notice that the View won’t be empty anymore, but it will display the fake data we’ve provided.

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/07/image_thumb1.png?resize=333%2C393" width="333" height="393"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/07/image1.png)

### 

### Wrapping up

As usual, I’ve commited all the samples described in this post in my GitHub project ([https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo](https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo "https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo")), inside the folder called **DesignData**. Have fun!