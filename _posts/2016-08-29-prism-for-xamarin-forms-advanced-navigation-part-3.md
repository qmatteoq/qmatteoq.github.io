---
id: 6608
title: 'Prism for Xamarin Forms &ndash; Advanced navigation (Part 3)'
date: 2016-08-29T16:00:00+00:00
author: qmatteoq
layout: post
guid: http://blog.qmatteoq.com/?p=6608
permalink: /prism-for-xamarin-forms-advanced-navigation-part-3/
categories:
  - Xamarin
tags:
  - MVVM
  - Prism
  - Xamarin
  - Xamarin Forms
---
In the previous post, we’ve expanded a bit our original sample application, by creating a service to interact with the APIs offered by the <a href="https://www.trackseries.tv/" target="_blank">TrackSeries</a> website, by using it to populate some data in the app and, in the end, by creating a detail page where to see more info about the selected show. This way, we have understood how Prism allows to manage some basic concepts like navigation, page’s lifecycle and dependency injection.

In this post we’re going to see a couple of more advanced concepts which, however, can be fundamental when it comes to develop a real project using Xamarin Forms.

### Advanced navigation

If you have tested the previous version of the app, you would have noted some issues with navigation, especially if you have navigated to the detail page of a TV Show. For example, if you test the UWP version on a Windows 10 PC, you will notice that the Back button that is usually available in the top left corner of the chrome of the windows is missing. Or on Android or iOS, the navigation bar which shows the title of the page and the virtual back button is missing, so if your device doesn’t have an actual back button (like an iPhone), you don’t have a way to go back to the home once you are into a show detail page.

[<img title="Screenshot_2016-08-20-22-51-58" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="Screenshot_2016-08-20-22-51-58" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-20-22-51-58_thumb.png?resize=290%2C515" width="290" height="515"  data-recalc-dims="1" />](https://i1.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-20-22-51-58.png)

If you have some previous experience with Xamarin Forms you should already have an idea why this problem is happening. Every basic page in a Xamarin Forms app is represented by the **ContentPage** class, but it can be embedded in other kind of pages to provide more advanced navigation scenarios, like a **NavigationPage** (to provide a navigation UI) or a **TabbedPage** (to show multiple pages in different tabs). In the sample we’ve created so far, we’ve just created two basic pages (of type **ContentPage**) and then we’ve simply navigated to them using the **NavigationService**. We haven’t specified anywhere that the pages should have been actually embedded into a **NavigationPage** to get access to the navigation UI.

To achieve this goal in a plain Xamarin Forms app, we would have done something like this:

<pre class="brush: csharp;">public partial class App : Application
{
    public App()
    {
        MainPage = new NavigationPage(new MainPage());
    }     
}
</pre>

An instance of the **MainPage** of the application is embedded into a new instance of a **NavigationPage**: from now on, the **NavigationPage** will be the container of each **ContentPage** that the user will see, providing a consistent navigation UI across every page of the app. However, with the Prism approach, we can’t recreate the same approach:

<pre class="brush: csharp;">public partial class App : PrismApplication
{
    public App(IPlatformInitializer initializer = null) : base(initializer) { }

    protected override void OnInitialized()
    {
        InitializeComponent();

        NavigationService.NavigateAsync("MainPage");
    }

    protected override void RegisterTypes()
    {
        Container.RegisterTypeForNavigation&lt;MainPage&gt;();
        Container.RegisterTypeForNavigation&lt;DetailPage&gt;();
        Container.RegisterType&lt;ITsApiService, TsApiService&gt;();
    }
}
</pre>

We have just registered, in the **Container**, the **MainPage** and then we have called the **NavigateAsync()** method of the **NavigationService**, which accepts only the key that identifies the destination page: we can’t specify, in any way, that this page should be embedded into a **NavigationPage**.

Luckily, Prism has a smart solution for this problem: deep linking. One of the features offered by Prism for Xamarin Forms, in fact, is to support complex queries as parameters of the **NavigateAsync()** method. For example, we can specify queries like “**MainPage/DetailPage?id=1”** to navigate directly to the detail page of the app and, at the same time, passing a parameter called **id** with value **1.** This approach is very useful when, for example, you want to link a specific page of your application from another application, a website or a section of your app.

<p align="left">
  We can leverage this feature also to achieve the goal of embedding our pages into a <strong>NavigationPage: </strong>first, we need to register the base <strong>NavigationPage </strong>type included in Xamarin Forms as a type for navigation in the <strong>Container</strong>. Then, we can use the query <strong>“NavigationPage/MainPage”</strong> to tell to the <strong>NavigationService</strong> that we need to navigate first to the page identified by the <strong>NavigationPage</strong> key and then to the one identified by the <strong>MainPage</strong> key. Since the <strong>NavigationPage </strong>isn’t actually a real page, but just a container, the end result will be the same we’ve seen in the first sample code: the <strong>MainPage</strong> (and every consequent page in the navigation flow) will be embedded into a <strong>NavigationPage.</strong>
</p>

Here is how our new **App** class looks like:

<pre class="brush: csharp;">public partial class App : PrismApplication
{
    public App(IPlatformInitializer initializer = null) : base(initializer) { }

    protected override void OnInitialized()
    {
        InitializeComponent();

        NavigationService.NavigateAsync("NavigationPage/MainPage");
    }

    protected override void RegisterTypes()
    {
        Container.RegisterTypeForNavigation&lt;NavigationPage&gt;();
        Container.RegisterTypeForNavigation&lt;MainPage&gt;();
        Container.RegisterTypeForNavigation&lt;DetailPage&gt;();
        Container.RegisterType&lt;ITsApiService, TsApiService&gt;();
    }
}
</pre>

&nbsp;

Thanks to this change, now we have a proper navigation bar, as you can see in the following screenshot taken from the Android version:

[<img title="Screenshot_2016-08-20-22-53-06" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="Screenshot_2016-08-20-22-53-06" src="https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-20-22-53-06_thumb.png?resize=290%2C516" width="290" height="516"  data-recalc-dims="1" />](https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-20-22-53-06.png)

Another example of complex deep linking is using query parameters. You can use a navigation query like the following one:

<pre class="brush: csharp;">NavigationService.NavigateAsync("FirstPage?id=1&title=First page");
</pre>

Automatically, the destination page will receive, in the **NavigationParams** object of the **OnNavigatedTo()** method, two items: one with key **id** and value **1** and one with key **title** and value **First page.**

<pre class="brush: csharp;">public void OnNavigatedTo(NavigationParameters parameters)
{
    string id = parameters["id"].ToString();
    string title = parameters["title"].ToString();

    Title = $"Page with id {id} and title {title}";
}
</pre>

****

Eventually, you can use this feature to leverage even more complex navigation flows, which involves container for multiple pages.

Let’s try to better understand this scenario with a real example. We have already talked about the concept that Xamarin Forms offers some pages which doesn’t display any actual content, but that they act as a container for other pages. We’ve already seen an example of this concept: the **NavigationPage** type doesn’t display any actual content, but it’s a container to add navigation UI and features to a **ContentPage**. Another similar container is **TabbedPage**, where every children page is displayed in a different tab.

Let’s say that we want to improve our TV show application and add two sections to the main page, using a **TabbedPage** control: the first section will display the list of upcoming shows, so it will be a simple **ContentPage;** the second section, instead, will display a list of the available TV Shows and, as such, it will be embedded into a **NavigationPage,** because we want to provide the ability to tap on a show and see more info about the selected show.

This is how our project would look like:

[<img title="prism4" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="prism4" src="https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/prism4_thumb.png?resize=430%2C319" width="430" height="319"  data-recalc-dims="1" />](https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/prism4.png)

The application has two main sections (**UpcomingShowsPage** and **ShowsListPage**) and a detail page (**DetailPage**), each of them with its own ViewModel. The main pages are presented to the user as two sections of a tab control, which is defined in the **MainTabbedPage.xaml** file:

<pre class="brush: csharp;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;TabbedPage xmlns="http://xamarin.com/schemas/2014/forms"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            xmlns:prism="clr-namespace:Prism.Mvvm;assembly=Prism.Forms"
            xmlns:views="clr-namespace:PrismTest.Views;assembly=PrismTest"
            prism:ViewModelLocator.AutowireViewModel="True"
            Title="Main page"
            x:Class="PrismTest.Views.MainTabbedPage"&gt;

  &lt;views:UpcomingShowsPage /&gt;
  &lt;NavigationPage Title="Shows list"&gt;
    &lt;x:Arguments&gt;
      &lt;views:ShowsListPage /&gt;
    &lt;/x:Arguments&gt;
  &lt;/NavigationPage&gt;

&lt;/TabbedPage&gt;
</pre>

The **UpcomingShowsPage** is a simple **ContentPage**, while the **ShowsListPsage** is embedded into a **NavigationPage**, since the user has the chance to move to the **DetailPage** to see more info about the selected TV show. Now let’s say that, as a consequence of an user action, we want to redirect the user to the detail page of a specific TV Show.&nbsp; With standard Xamarin Forms it wouldn’t a hard task to accomplish, but the real challenge would be to retain the whole navigation stack: we want to bring the user to the detail page, but we also want that, when he presses the back button, he follows the proper backward navigation flow (so **DetailPage –> ShowListPage**). Additionally, everything should be done by keeping the focus in the second tab, since **ShowListPage** is part of a **TabbedPage**.

Sounds complicated, isn&#8217;t it? Well, here is how it’s easy to achieve this goal with Prism:

<pre class="brush: csharp;">public partial class App : PrismApplication
{
    public App(IPlatformInitializer initializer = null) : base(initializer) { }

    protected override void OnInitialized()
    {
        InitializeComponent();

        NavigationService.NavigateAsync("MainTabbedPage/NavigationPage/ShowsListPage/DetailPage?id=1");
    }

    protected override void RegisterTypes()
    {
        Container.RegisterTypeForNavigation&lt;UpcomingShowsPage&gt;();
        Container.RegisterTypeForNavigation&lt;ShowsListPage&gt;();
        Container.RegisterTypeForNavigation&lt;DetailPage&gt;();
        Container.RegisterTypeForNavigation&lt;MainTabbedPage&gt;();
        Container.RegisterTypeForNavigation&lt;NavigationPage&gt;();
    }
}
</pre>

<pre class="brush: csharp;"></pre>

As usual, in the **RegisterTypes()** method, we have registered every page that compose our application. Then, we invoke the **NavigateAsync()** method of the **NavigationService** passing the whole path we want to follow: **MainTabbedPage/NavigationPage/ShowsListPage/DetailPage** with an additional parameter that identifies the selected TV Show, that we can intercept in the **OnNavigatedTo()** method of the **DetailPageViewModel**.

<pre class="brush: csharp;">public class DetailPageViewModel : BindableBase, INavigationAware
{
    private readonly ITsApiService _tsApiService;
    private SerieInfoVM _selectedShow;

    public SerieInfoVM SelectedShow
    {
        get { return _selectedShow; }
        set { SetProperty(ref _selectedShow, value); }
    }

    public DetailPageViewModel(ITsApiService tsApiService)
    {
        _tsApiService = tsApiService;
    }

    public void OnNavigatedFrom(NavigationParameters parameters)
    {

    }

    public async void OnNavigatedTo(NavigationParameters parameters)
    {
        int id = Convert.ToInt32(parameters["id"]);
        SelectedShow = await _tsApiService.GetSerieById(id);
    }
}
</pre>

Thanks to Prism, other than achieving the goal of redirecting the user directly to the page we’re interested into, we have also retained the full backward navigation stack. The following images show you what happens when you press the Back button:

<table cellspacing="0" cellpadding="2" width="400" border="0">
  <tr>
    <td valign="top" width="200">
      <a href="https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-22-11-40-17.png"><img title="Screenshot_2016-08-22-11-40-17" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="Screenshot_2016-08-22-11-40-17" src="https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-22-11-40-17_thumb.png?resize=290%2C516" width="290" height="516"  data-recalc-dims="1" /></a>
    </td>
    
    <td valign="top" width="200">
      <a href="https://i1.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-22-11-42-41.png"><img title="Screenshot_2016-08-22-11-42-41" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="Screenshot_2016-08-22-11-42-41" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-22-11-42-41_thumb.png?resize=290%2C516" width="290" height="516"  data-recalc-dims="1" /></a>
    </td>
  </tr>
</table>

As you can see, from the detail page of the show (the screenshot on the left) we’ve been properly redirected to the previous page in the stack (the shows list, displayed in the screenshot on the right), even if we didn’t actually visited it during our navigation flow (since the app was directly loaded in the detail page). Additionally, we’ve kept the focus on the second tab (**Shows list**) so the user has still the chance, at any time, to move to the first one (**Upcoming shows**). Pretty cool, isn’t it?

**Attention**: in the current Prism implementation dealing with the **TabbedPage** control has a downside. In our example, as we’ve seen from the screenshot of the project’s structure, the Upcoming Shows section is represented by a standard page (**UpcomingShowsPage**) with its own ViewModel (**UpcomingShowsPageViewModel**), which implements the **INavigationAware** interface with the goal to leverage the **OnNavigatedTo()** method to load the data (in our case, the list of upcoming shows). As such, the **UpcomingShowsPageViewModel** would look like this:

<pre class="brush: csharp;">public class UpcomingShowsPageViewModel : BindableBase, INavigationAware
{
    private readonly ITsApiService _tsApiService;

    private ObservableCollection&lt;SerieFollowersVM&gt; _topSeries;

    public ObservableCollection&lt;SerieFollowersVM&gt; TopSeries
    {
        get { return _topSeries; }
        set { SetProperty(ref _topSeries, value); }
    }

    public UpcomingShowsPageViewModel(ITsApiService tsApiService)
    {
        _tsApiService = tsApiService;
    }

    public void OnNavigatedFrom(NavigationParameters parameters)
    {

    }

    public async void OnNavigatedTo(NavigationParameters parameters)
    {
        var series = await _tsApiService.GetStatsTopSeries();
        TopSeries = new ObservableCollection&lt;SerieFollowersVM&gt;(series);
    }
}
</pre>

However, if you tap on the **Upcoming Shows** tab you’ll notice that nothing won’t happen and the **OnNavigatedTo()** method won’t be triggered. The reason is that the navigation methods implemented by the **INavigationAware** interface are raised only when you navigate using the Prism **NavigationService**. If the navigation happens without leveraging it (like, in this case, where the navigation to the other tab is handled directly by the Xamarin Forms infrastructure), the **OnNavigatedTo()** method in the ViewModel will never be invoked and, as such, our data will never be loaded. There’s a solution in the works, which involves using a behavior, but it hasn’t been included yet in the current Prism version. You can follow the discussion and the proposed solution on GitHub: [https://github.com/PrismLibrary/Prism/issues/650](https://github.com/PrismLibrary/Prism/issues/650 "https://github.com/PrismLibrary/Prism/issues/650")

## 

### Wrapping up

In this post we’ve learned how to leverage the deep linking featured offered by Prism, which allows to handle complex navigation patterns in an easy way, keeping at the same time the proper backward navigation path. In the next post (which will be the last one), we’ll see instead how to use platform specific code in a Xamarin Forms application created with Prism. You can find all the samples on my GitHub repository: the InfoSeries one ([https://github.com/qmatteoq/XamarinForms-Prism/tree/master/InfoSeries](https://github.com/qmatteoq/XamarinForms-Prism/tree/master/InfoSeries "https://github.com/qmatteoq/XamarinForms-Prism/tree/master/InfoSeries")) shows you the first approach (simple navigation using a NavigationPage), the DeepNavigation one ([https://github.com/qmatteoq/XamarinForms-Prism/tree/master/DeepNavigation](https://github.com/qmatteoq/XamarinForms-Prism/tree/master/DeepNavigation "https://github.com/qmatteoq/XamarinForms-Prism/tree/master/DeepNavigation")) instead shows you the advanced deep link feature we’ve seen in the second part of the post. Happy coding!