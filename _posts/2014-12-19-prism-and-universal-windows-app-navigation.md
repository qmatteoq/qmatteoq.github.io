---
id: 6322
title: 'Prism and Universal Windows app &ndash; Navigation'
date: 2014-12-19T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6322
permalink: /prism-and-universal-windows-app-navigation/
categories:
  - Universal Apps
tags:
  - Prism
  - Windows 8
  - Windows Phone
  - wpdev
---
Let’s continue our journey about using Prism in Universal Windows app development, by understanding how to manage a very important feature: navigation. Unless we’re talking about a really basic application, your project will likely contain more than one page. Consequently, we need to be able to navigate from one page to another. The problem is that, typically, the operation is performed using the **Frame** class, which can be accessed only in code behind, since the pages inherits from base **Page** class. To achieve the same result in a MVVM application, we need to use a special service that act as a wrapper of the **Frame** class and that can be accessed also from a ViewModel. Prism already offers this class: it’s called **NavigationService** and we’ve already seen <a href="http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/" target="_blank">it in the first post</a>, when we’ve setup the infrastructure required by Prism to properly work. If you remember, the **App.xaml.cs** file contains the following method:

<pre class="brush: csharp;">protected override Task OnLaunchApplicationAsync(LaunchActivatedEventArgs args)
{
    NavigationService.Navigate("Main", null);
    return Task.FromResult&lt;object&gt;(null);
}
</pre>

This method is invoked when the application is started and it’s used to redirect the user the user to the main page of the application. We’ve already described how to perform a basic navigation using the **NavigationService**: we call the **Navigate()** method, passing as parameter a string with the name of the View, without the **Page** suffix. In the previous sample, passing **Main** as parameter of the **Navigate()** method means that we want to redirect the user to a View in our project that is called **MainPage.xaml**.

### Using the NavigationService in a ViewModel

To use the **NavigationService** in a ViewModel we need to use the dependency injection approach we’ve already seen in the first post. The **App.xaml.cs** file, in fact, contains also the following method:

<pre class="brush: csharp;">protected override Task OnInitializeAsync(IActivatedEventArgs args)
{
    // Register MvvmAppBase services with the container so that view models can take dependencies on them
    _container.RegisterInstance&lt;ISessionStateService&gt;(SessionStateService);
    _container.RegisterInstance&lt;INavigationService&gt;(NavigationService);
    // Register any app specific types with the container

    // Set a factory for the ViewModelLocator to use the container to construct view models so their 
    // dependencies get injected by the container
    ViewModelLocationProvider.SetDefaultViewModelFactory((viewModelType) =&gt; _container.Resolve(viewModelType));
    return Task.FromResult&lt;object&gt;(null);
}
</pre>

As you can see, among other things, the method takes of registering, inside the **UnityContainer** object, the **INavigationService** interface, by connecting it to the **NavigationService** implementation offered by Prism. This way, we’ll be able to use the **NavigationService** in a ViewModel simply by adding an **INavigationService** parameter to the ViewModel’s constructor, like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private readonly INavigationService _navigationService;

    public MainPageViewModel(INavigationService navigationService)
    {
        _navigationService = navigationService;
    }
}

</pre>

Now we have a reference to the Prism’s **NavigationService**, that we can use to perform navigation inside a ViewModel.

### Managing the navigation events in a ViewModel

Another common requirement when you develop an Universal Windows app using the MVVM pattern is to find a way to intercept when the user is navigating to or away from the current page. In code behind, it’s easy to do it because we have access to two methods called **OnNavigatedTo()** and **OnNavigatedFrom()**: unfortunately, they are available only in the code behind class, since they are inherited from the base **Page** class.

Prism offers a simple way to manage these two navigation events in a ViewModel: as we’ve already seen in the previous posts, the ViewModels inherits from a base class called **ViewModel**. Thanks to this class, we are able to subscribe to the **OnNavigatedTo()** and **OnNavigatedFrom()** methods inside a ViewModel, like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    public override void OnNavigatedTo(object navigationParameter, NavigationMode navigationMode, Dictionary&lt;string, object&gt; viewModelState)
    {
        base.OnNavigatedTo(navigationParameter, navigationMode, viewModelState);
    }

    public override void OnNavigatedFrom(Dictionary&lt;string, object&gt; viewModelState, bool suspending)
    {
        base.OnNavigatedFrom(viewModelState, suspending);
 
    }
}

</pre>

The **OnNavigatedTo()** method, especially, is very important, because it’s the best point where to load the data that will be displayed in the page. Some people, in fact, often use the ViewModel constructor to load the data: however, it’s not a good approach, especially in modern development. It often happens, in fact, that the data needs to be loaded using asynchronous methods, which are based on the async and await pattern. If you have some experience with this approach, you’ll know that a class constructor can’t be marked with the **async** keyword, so you won’t be able to call a method using the **await** prefix. This means that, for example, the following code won’t compile:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private readonly INavigationService _navigationService;
    private readonly IFeedService _feedService;
    private ObservableCollection&lt;News&gt; _news;

    public ObservableCollection&lt;News&gt; News
    {
        get { return _news; }
        set { SetProperty(ref _news, value); }
    }

    public MainPageViewModel(INavigationService navigationService, IFeedService feedService)
    {
        _navigationService = navigationService;
        _feedService = feedService;

        IEnumerable&lt;News&gt; news = await _feedService.GetNews();

        News = new ObservableCollection&lt;News&gt;();
        foreach (News item in news)
        {
            News.Add(item);
        }
    }        
}
</pre>

I won’t describe in details how is defined the **IFeedService** class and how exactly works the **GetNews()** method: you can see all the details in the source code of the project that is published at [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample"). For the moment, it’s important just to know that it’s an asynchronous method which, by using the **SyndicationClient** class provided by the Windows Runtime, downloads the RSS feed of this blog and parses it, to return the items as list of objects. Our goal is to display this list using a **ListView** control in the application: however, as I’ve previously mentioned, the previous code won’t compile, since I’m calling an asynchronous method (**GetNews()**, which is invoked with the **await** keyword) inside the ViewModel constructor, which can’t be marked with the **async** keyword.

The **OnNavigatedTo()**, instead, since it’s a sort of event handler (it manages the page navigation event), can be marked with the **async** keyword, so you can call asynchronous method in it without problems. Here is the correct approach to implement the previous sample:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private readonly INavigationService _navigationService;
    private readonly IFeedService _feedService;
    private ObservableCollection&lt;News&gt; _news;

    public ObservableCollection&lt;News&gt; News
    {
        get { return _news; }
        set { SetProperty(ref _news, value); }
    }

    public MainPageViewModel(INavigationService navigationService, IFeedService feedService)
    {
        _navigationService = navigationService;
        _feedService = feedService;
    }

    public override async void OnNavigatedTo(object navigationParameter, NavigationMode navigationMode, Dictionary&lt;string, object&gt; viewModelState)
    {
        IEnumerable&lt;News&gt; news = await _feedService.GetNews();

        News = new ObservableCollection&lt;News&gt;();
        foreach (News item in news)
        {
            News.Add(item);
        }
    }
}

</pre>

As you can see, the data loading operation is now performed in the **OnNavigatedTo()** method, which is marked with the **async** keyword and, consequently, we can call the **GetNews()** method using the **await** prefix without any issue. This code will compile and run just fine!

### Passing parameters from one page to another

The **OnNavigatedTo()** can be useful also in another scenario: to pass parameters from one page to the another. Let’s use again the previous sample and let’s say that we have the following XAML page, which is the same we’ve seen in the <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-advanced-commands/" target="_blank">previous post</a> talking about commands with parameters:

<pre class="brush: xml;">&lt;storeApps:VisualStateAwarePage
    x:Class="Prism_Navigation.Views.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Prism_Navigation"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:storeApps="using:Microsoft.Practices.Prism.StoreApps"
    xmlns:mvvm="using:Microsoft.Practices.Prism.Mvvm"
    xmlns:interactivity="using:Microsoft.Xaml.Interactivity"
    xmlns:core="using:Microsoft.Xaml.Interactions.Core"
    mc:Ignorable="d"
    mvvm:ViewModelLocator.AutoWireViewModel="True"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

    &lt;Grid&gt;
        &lt;ListView ItemsSource="{Binding Path=News}" SelectionMode="Single" IsItemClickEnabled="True" Margin="12, 0, 12, 0"&gt;
            &lt;ListView.ItemTemplate&gt;
                &lt;DataTemplate&gt;
                    &lt;StackPanel&gt;
                        &lt;TextBlock Text="{Binding Title}" Style="{StaticResource SubheaderTextBlockStyle}" 
                       TextWrapping="Wrap" /&gt;
                    &lt;/StackPanel&gt;
                &lt;/DataTemplate&gt;
            &lt;/ListView.ItemTemplate&gt;
            &lt;interactivity:Interaction.Behaviors&gt;
                &lt;core:EventTriggerBehavior EventName="ItemClick"&gt;
                    &lt;core:InvokeCommandAction Command="{Binding Path=ShowDetailCommand}" /&gt;
                &lt;/core:EventTriggerBehavior&gt;
            &lt;/interactivity:Interaction.Behaviors&gt;
        &lt;/ListView&gt;
    &lt;/Grid&gt;
&lt;/storeApps:VisualStateAwarePage&gt;

</pre>

We’re using a **ListView** control to display the list of news retrieved by the **GetNews()** method of the **FeedService** class. By using the behavior we’ve learned to use in the previous post, we’ve connected the **ItemClick** event to the **ShowDetailCommand** command in the ViewModel. Here is how the **ShowDetailCommand**’s definiton looks like:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    public MainPageViewModel(INavigationService navigationService, IFeedService feedService)
    {
        _navigationService = navigationService;
        _feedService = feedService;

        ShowDetailCommand = new DelegateCommand&lt;ItemClickEventArgs&gt;((args) =&gt;
        {
            News selectedNews = args.ClickedItem as News;
            _navigationService.Navigate("Detail", selectedNews);
        });
    }

    public DelegateCommand&lt;ItemClickEventArgs&gt; ShowDetailCommand { get; private set; }
}

</pre>

The approach is the same we’ve seen in the previous post: we’ve defined a **DelegateCommand<ItemClickEventArgs>** and, consequently, we are able to retrieve, in the command definition, the item selected by the user. The difference is that, this time, after casting it as a **News** object, we pass it as second parameter of the **Navigate()** method of the **NavigationService**. This way, other than redirecting the user to a page called **DetailPage.xaml** (since we’re using the string **Detail**), we bring the information about the selected news, so that we can show the details.

The **OnNavigatedTo()** method can be used also to retrieve the parameter that we’ve passed from the **NavigationService**, thanks to one of the parameters called **navigationParameter**. The following sample shows the definition of the ViewModel of the detail page, which is the destination page of the navigation:

<pre class="brush: csharp;">public class DetailPageViewModel : ViewModel
{
    private News _selectedNews;

    public News SelectedNews
    {
        get { return _selectedNews; }
        set { SetProperty(ref _selectedNews, value); }
    }

    public override void OnNavigatedTo(object navigationParameter, NavigationMode navigationMode, Dictionary&lt;string, object&gt; viewModelState)
    {
        if (navigationParameter != null)
        {
            SelectedNews = navigationParameter as News;
        }
    }
}
</pre>

As you can see, thanks to the **navigationParameter**, we are able to retrieve the selected item that has been passed by the previous page. In the sample, we simply cast it back to the **News** type (since it’s a generic object) and we display it to the user with the following XAML:

<pre class="brush: xml;">&lt;Grid&gt;
    &lt;StackPanel&gt;
        &lt;TextBlock Text="{Binding Path=SelectedNews.Title}" /&gt;
        &lt;TextBlock Text="{Binding Path=SelectedNews.Summary}" /&gt;
    &lt;/StackPanel&gt;
&lt;/Grid&gt;
</pre>

### Managing the back button in Windows Phone 8.1

One of the most important differences in the navigation system between Windows Phone 8.0 and Windows Phone 8.1 is the back button management: by default, in Windows Phone 8.0, the Back button always redirects the user to the previous page of the application. In Windows Phone 8.1, instead, to keep the behavior consistent with Windows 8.1 (which doesn’t offer a hardware button), the Back button redirect the user to the previous application. However, Microsoft don’t suggest to developers to use this approach: users, by pressing the back button, expect to go back to the previous page of the application since Windows Phone 7.0. Consequently, you need to override, in every page or in the App.xaml.cs, the **HardwareButtons.BackPressed** event and to perform a similar code:

<pre class="brush: csharp;">private void HardwareButtons_BackPressed(object sender, BackPressedEventArgs e)
{
    Frame frame = Window.Current.Content as Frame;
    if (frame == null)
    {
        return;
    }

    if (frame.CanGoBack)
    {
        frame.GoBack();
        e.Handled = true;
    }
}

</pre>

The code takes care of checking if there are pages in the backstack of the application (by checking the value of the **CanGoBack** property): if this is the case, we call the **GoBack()** method to perform a navigation to the previous page and we set the **Handled** property of the event handler as **true**, so that we prevent the operating system to manage it.

Well, the good news is that the **MvvmAppBase** class, which is the one that replaces the **App** one in a Prism application, already takes care of this for us: we won’t have to do nothing or to write additional code to support a proper Back key management in a Windows Phone 8.1 application.

### Wrapping up

As usual, you can download the sample code related to this post from GitHub at the following URL: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")

### Index of the posts about Prism and Universal Windows apps

  1. [The basic concepts](http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/ "Prism and Universal Windows app – The basic concepts")
  2. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-binding-and-commands/" target="_blank">Binding and commands</a>
  3. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-advanced-commands" target="_blank">Advanced commands</a>
  4. Navigation
  5. [Managing the application&#8217;s lifecycle](http://wp.qmatteoq.com/prism-and-universal-windows-app-managing-the-applications-lifecycle/ "Prism and Universal Windows app – Managing the application’s lifecycle")
  6. [Messages](http://wp.qmatteoq.com/prism-and-universal-windows-apps-messages/ "Prism and Universal Windows apps – Messages")
  7. [Layout management](http://wp.qmatteoq.com/prism-and-universal-windows-apps-layout-management/ "Prism and Universal Windows apps – Layout management")

Sample project: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")