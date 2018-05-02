---
id: 6259
title: 'Using Caliburn Micro with Universal Windows app &ndash; Navigation'
date: 2014-07-02T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6259
permalink: /using-caliburn-micro-with-universal-windows-app-navigation/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Caliburn
  - Windows 8
  - Windows Phone
---
Let’s continue our journey with Caliburn Micro and Universal Windows app. After talking about the <a href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-the-project-setup/" target="_blank">project setup</a> and <a href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-binding-actions" target="_blank">binding and actions</a>, let’s see how to manage navigation in a Windows and Windows Phone 8.1 app.

### The NavigationService

Managing navigation is one of the challengers offered by the MVVM pattern: the Windows Runtime offers a static class called **Frame**, which exposes the methods needed to perform the navigation. This class is available only in the code behind, since it’s inherited from the **Page** class, which is the base class from which every application’s page inherits from. Since the ViewModels don’t inherit from the **Page** class, you won’t be able to directly access to the **Frame** class. Here comes the **NavigationService** class, which is a wrapper that can be used to perform navigation duties inside a ViewModel. Using it is really simple: since it’s one of the services that is embedded in Caliburn Micro, it’s automatically registered during the startup (do you remember the **RegisterWinRTServices()** method that is called in the **Configure()** method defined in th **App.xaml.cs** file?).

To get access to the **NavigationService**, you simply need to add a parameter of type **INavigationService** to the ViewModel constructor: the container will take care of injecting the concrete implementation of the service at runtime. Here is a sample of a ViewModel that registers the NavigationService:

<pre class="brush: csharp;">public sealed class MainPageViewModel: Screen
{
    private readonly INavigationService _navigationService;

    public MainPageViewModel(INavigationService navigationService)
    {
        this._navigationService = navigationService;
    }
}</pre>

Now you’ll be able to use the **NavigationService** across the ViewModel: for example, you can use it in a method that is triggered when a button is clicked. One of the nice things of the **NavigationService** class is that supports a **ViewModel-First** approach, which is very useful when you work with the MVVM: you can specify, instead of the page where to redirect the user, the ViewModel that is associated to the page. Caliburn Micro will take care of resolving the correct page and it will perform the navigation. This goal is achieved using the method **NavigateToViewModel<T>()**, where **T** is the type of ViewModel where we want to redirect the user.

Here is a sample:

<pre class="brush: csharp;">public void GoToDetail()
{
    _navigationService.NavigateToViewModel&lt;DetailPageViewModel&gt;();
}</pre>

### Managing parameters

One of the most interesting changes included in the Windows Runtime is that navigation isn’t based anymore on uris like in Silverlight: as a consequence, you are able to pass as navigation’s parameters also complex objects, while in Windows Phone 8.0 you were forced to pass only plain data (like texts or numbers). Caliburn Micro 2.0 supports this scenario, simply by accepting a parameter in the **NavigateToViewModel<T>()** method. Let’s use the same sample we’ve seen <a href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-binding-actions" target="_blank">in the previous post</a>: we’re going to display a list of movies using a **ListView** control. When the user taps on one of them, we’re going to use the **ItemClick** event to redirect him on the detail page.

<pre class="brush: csharp;">public void GoToDetail(Movie movie)
{
    _navigationService.NavigateToViewModel&lt;DetailPageViewModel&gt;(movie);
}</pre>

We’ve simply added the object received by the method **GoToDetail()** as parameter to the **NavigateToViewModel<T>()** method. Now we have another problem: how to get the parameter in the destination ViewModel? In an application developed without MVVM, we would have used the **OnNavigatedTo()** method defined in the code behind: the parameter returned by the method contains a property, called **Parameter**, with the object that has been sent by the source page. But, again, this is a scenario that can be satisfied because the code behind class inherits from the **Page** class: we can’t say the same for the ViewModel class.

Caliburn Micro uses a naming convention to manage this scenario: you just have to define, in the destination ViewModel, a property called **Parameter**, which type is the same of the object you’ve passed to the **NavigateToViewModel<T>()** method. Since, in the previous example, we passed a **Movie** object, here is how we can setup the **DetailPageViewModel** class:

<pre class="brush: csharp;">public class DetailPageViewModel: Screen
{
    public Movie Parameter { get; set; }

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

    protected override void OnActivate()
    {
        Title = Parameter.Title;
    }
}</pre>

As you can see, in the **DetailPageViewModel** class we’ve added a new public property called **Parameter**, which type is **Movie**. This way, we’ll be able to access to the **Movie** object sent by the main page: in this sample, we’re going to display the title of the movie in the page , by assigning it to the **Title** property. The operation is performed in the **OnActivate()** method, which is triggered when the page is displayed: it’s one of the navigation events that is offered by the **Screen** class which, as you may have noticed, is the one all the ViewModels are inheriting from. With this method, we are able to recreate the **OnNavigatedTo()** event inside the ViewModel.

### Managing the back button

One of the biggest differences between Windows Phone 8.0 and Windows Phone 8.1 is the back button management: as a consequence of the alignment with the Windows 8 platform (which doesn’t offer a hardware back key button), the default behavior when the Back button is pressed is to redirect the user to the previous application and not to the previous page. This doesn’t mean that it’s correct, from a user experience point of view, to keep this behavior: the user expects to go back to the previous page of your application when the Back button is pressed. To support the developer to correctly manage this scenario, all the Visual Studio templates for the Universal Windows apps (except the Blank App one) include a class called **NavigationHelper**, which, among other things, automatically intercepts the Back button pressed event and redirects the user to the previous page of the app.

However, if you’re using Caliburn Micro, you won’t need it: the framework will take care, automatically, of managing the back button for you. In the previous sample, you’ll notice the by pressing the Back button in the detail page, you’ll be correctly redirected to the main page of the app and not the previous application in the OS stack.

### Wrapping up

In this post we’ve learned how to properly manage navigation in a Universal Windows app using Caliburn Micro. As usual, you can download a sample project to play with <a href="http://1drv.ms/1nZN5qM" target="_blank">from the following link</a>.

### Caliburn Micro 2.0 in Universal Windows apps &#8211; The complete series

  1. [The project setup](http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-the-project-setup/ "Using Caliburn Micro with Universal Windows app – The project setup")
  2. <a title="Using Caliburn Micro with Universal Windows app – Binding and actions" href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-binding-actions/" target="_blank">Binding and actions</a>
  3. Navigation

<a href="https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo" target="_blank">Samples available on GitHub</a>