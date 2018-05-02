---
id: 2112
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Using launchers and choosers'
date: 2013-03-12T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2112
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
In the <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-messaging/" target="_blank">previous post</a> we’ve seen how the messaging infrastructure works in Caliburn Micro. In this post we’ll see how to use launchers and choosers within a ViewModel: the two topics are strictly connected, since to accomplish this task Caliburn uses a special type of messages. In fact, when we want to interact with launchers and choosers, we exchange messages using the **IEventAggregator** class we learned to use in the previous post. The difference is that messages are “hidden” and already built in in the toolkit, so we won’t need to define them. Let’s start!

I assume that, since you’re reading this blog and since MVVM is an “advanced “ topic, you already know what launchers and choosers are. In case not, it’s a way that the operating system offers to interact with native applications: for example, displaying a position in the native Map app; or import a contact from the People hub; or send a SMS using the Messages app.

### Launchers

Let’s see how to interact with launchers, that are the tasks that are used to demand an operation to a native application, without expecting anything in return: we’ll do this by using the **MapsTask** class, that is used to display a location in the native maps application. I assume that you already have an application that is created using the Caliburn toolkit, so that you have a View connected to a ViewModel.

In the View we add a button, that we will use to execute the launcher:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;Button Content="Launch map" x:Name="LaunchMap" /&gt;
&lt;/StackPanel&gt;</pre>

As usual, thanks to the Caliburn naming conventions, the click on the button will trigger the **LaunchMap** method in the ViewModel. Let’s take a look at the ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel: Screen
{
    public MainPageViewModel(IEventAggregator eventAggregator)
    {
        this.eventAggregator = eventAggregator;
        eventAggregator.Subscribe(this);
    }

    public void LaunchMap()
    {
        eventAggregator.RequestTask&lt;MapsTask&gt;(task =&gt;
                                                  {
                                                      task.SearchTerm = "Milan";
                                                  });
    }
}</pre>

As we’ve seen in the <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-messaging/" target="_blank">previous post</a>, it’s enough to add in the constructor a parameter which type is **IEventAggregator** to have a reference to the real object in our ViewModel. The first step is to subscribe to receive messages, by using the **Subscribe** method, passing the value **this** as parameter (since we want to register the ViewModel itself).

In the **LaunchMap** method we use again the **EventAggregator**, by calling an extension method that is available exclusively for Windows Phone: **RequestTask<T>**, where **T** is the launcher we want to execute (selected by one of the objects that is available in the **Microsoft.Phone.Tasks** namespace). As parameter of the method we can pass a delegate (in this sample, we use a lambda expression): this is needed for all the launchers or choosers that need some properties to be set to work properly. In this sample, we set the **SearchTerm** property of the task, to specify the location we want to search on the map. If we’re going to use a launcher that doesn’t need any parameter (like the **MarketplaceDetailTask**), we can simply pass nothing to the method.

If you try this code on the emulator or on the phone, you’ll see the Maps app opening and locating the Milan position as soon as you’ll tap on the button.

### 

### Choosers

Choosers work in a little bit different mode: the way we’re going to execute it is the same that we’ve seen for launchers but, in this case, we expect also a result in return, that we need to manage. As I’ve already told you, launchers and choosers infrastructure in Caliburn is based on messaging, so the way we’re going to manage results is the same we’ve seen in the previous post to manage incoming messages. Let’s see how it works using the **PhoneNumberChooserTask**, that can be used to import a contact from the People hub to the application.

First, let’s add a new button in the XAML:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;Button Content="Open contact" x:Name="OpenContact" /&gt;
&lt;/StackPanel&gt;</pre>

Now let’s take a look at the ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel: Screen, IHandle&lt;TaskCompleted&lt;PhoneNumberResult&gt;&gt;
{
    private readonly IEventAggregator eventAggregator;

    public MainPageViewModel(IEventAggregator eventAggregator, INavigationService navigationService)
    {
        this.eventAggregator = eventAggregator;

        eventAggregator.Subscribe(this);
    }

    public void OpenContact()
    {
        eventAggregator.RequestTask&lt;PhoneNumberChooserTask&gt;();
    }

    public void Handle(TaskCompleted&lt;PhoneNumberResult&gt; message)
    {
        MessageBox.Show(message.Result.DisplayName);
    }
}</pre>

The way a chooser works is the same as launchers: we call the **RequestTask<T>** method of the **EventAggregator**, passing as value of the T parameter the chooser’s type (**PhoneNumberChooserTask**). In this case, we don’t pass any parameter, since the chooser doesn’t require any property to be set to work properly. To manage, instead, the return value we use the same approach we’ve seen to receive a message: we inherit our ViewModel from the classe **IHandle<T>**, where T is **TaskCompleted<T>**. In this case, **T** is the class used by the operatying system to return a chooser’s result to the caller: in this sample, it’s a **PhoneNumberResult**, since it contains the phone number of the selected contact.

Once we inherit the ViewModel from this class, we need to implement the **Handle** method, that receives as a parameter the result returned by the chooser. It’s up to us what to do: in this sample, we simply take the **DisplayName** property (which contains the name of the selected contact) and we show it using a **MessageBox**.

### How to correctly manage the launchers and choosers lifecycle

There’s a problem in using the messaging architecture to manage launchers and choosers. When you subscribe a ViewModel to wait for incoming messages using the **Subscribe** method of the **EventAggregator,** the subscription is kept alive until the ViewModel instance itself is alive. In case of launchers and choosers this can cause some issues: if you rememer in the previous post, we were able to exchange messages between two different ViewModels (more precisely, it’s the reason why messages are used); in this case, instead, we’re using the messaging infrastructure to exchange messages inside a single ViewModel. What happens in the end is that if, for example, you move the **OpenContact** method to another ViewModel, but you keep the MainPageViewModel registered to listen for incoming **TaskCompleted<PhoneNumberResult>** message, you will notice that the MainPageViewModel will answers to requests that are triggered by other view models. This can cause some problems, especially if you need to use the same launcher or chooser in multiple views, so you have many view models registered to receive the same message.

This is valid especially for the MainViewModel: since it’s connected to the first page of the application, its instance is always alive until the app is closed, unlike for the other pages of the application (if you’re in the second page of your application and you go back to the main page, the second page is dismissed and its ViewModel is disposed).

The best way to manage this situation is to use the navigation events we’ve learned to use in <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">a previous post</a> and to inherit our ViewModel from the **Screen** class: this way we can call the **Subscribe** method of the **EventAggregator** class when the user navigate towards the page (managed by the **OnActivate** method) and call the **Unsubscribe** method, instead, when the user leaves the page (managed by the **OnDeactivate** method). With this approach, when the user navigates to another page the previous page isn’t subscribed anymore to receive messages and, as a consequence, will ignore any interaction with launchers and choosers.

Here is a sample of the updated ViewModel

<pre class="brush: csharp;">public class MainPageViewModel: Screen, IHandle&lt;TaskCompleted&lt;PhoneNumberResult&gt;&gt;
{
    private readonly IEventAggregator eventAggregator;

    public MainPageViewModel(IEventAggregator eventAggregator)
    {
        this.eventAggregator = eventAggregator;
    }
    protected override void OnActivate()
    {
        eventAggregator.Subscribe(this);
        base.OnActivate();
    }

    protected override void OnDeactivate(bool close)
    {
        eventAggregator.Unsubscribe(this);
        base.OnDeactivate(close);
    }

    public void OpenContact()
    {
        eventAggregator.RequestTask&lt;PhoneNumberChooserTask&gt;();
    }

    public void Handle(TaskCompleted&lt;PhoneNumberResult&gt; message)
    {
        MessageBox.Show(message.Result.DisplayName);
    }
}</pre>

As you can see, we don’t call anymore the **Subscribe** method of the **EventAggregator** class in the ViewModel constructor, but we’ve moved it in the **OnActivate** method.

Using the link below ou can downlod a sample project if you want to play with launchers and choosers on your own.

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:f6b43e35-360e-4b33-9614-69fe2a6cffa4" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_LaunchersChoosers.zip" target="_blank">Download the sample project</a>
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
                              * Using launchers and choosers 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>