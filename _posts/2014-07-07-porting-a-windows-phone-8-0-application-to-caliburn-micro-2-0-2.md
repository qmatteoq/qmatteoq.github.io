---
id: 6274
title: Porting a Windows Phone 8.0 application to Caliburn Micro 2.0
date: 2014-07-07T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6274
permalink: /porting-a-windows-phone-8-0-application-to-caliburn-micro-2-0-2/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Caliburn
  - Windows 8
  - Windows Phone
---
In my everyday job at Funambol I work on an application called <a href="http://www.windowsphone.com/s?appid=e54a9f5a-6ba8-43d9-a62b-bf14606d7e76" target="_blank">OneMediaHub</a>, which is the Windows Phone client for the cloud services offered by the company. The application is built using the MVVM pattern and Caliburn Micro as a framework. Now that the development cycle of the new version of the client is completed, one of my goal before starting working on the new features for the next version is to upgrade Caliburn Micro from the previous version (1.5.3) to the most recent one (2.0).

However, as I discovered by my self, the procedure isn’t so straight forward: Caliburn Micro is a big improvemenet compared to the previous version, but it also includes many breaking changes. In this post I’ll detail the most important ones I had to face during the migration.

### Changes in bootstrapper

The boostrapper is the base class that replaces the **App** one and that takes care of initializing the app, among all the Caliburn services and conventions.

There are two important changes in the boostrapper with Caliburn Micro 2.0:

  * In the previous release, the bootstrapper’s class had to inherit from the **PhoneBootstrapper** one. Now, instead, the base class has been renamed to **PhoneBootstrapperBase**.
  * Now, in the constructor of the bootstrapper’s class, you need to call the **Initialize()** method, while in the previous version it wasn’t required.

Here is a full working initialization of the bootstrapper for Caliburn Micro 2.0:

<pre class="brush: csharp;">public class CaliburnBootStrapper : PhoneBootstrapperBase
{
    public OneMediaHubBootStrapper()
    {
        this.Initialize();
    }
}
</pre>

Thanks to Matteo Tumiat that pointed me in the right direction: without calling the **Initialize()** method, the app was stuck in the loading phase, without any warning or exception.

### Changes in EventAggregator

**EventAggregator** is the class used to manage messages (which are, in the end, simple objects) that can be sent and received from different classes (like two ViewModels or a ViewModel and a View). This approach helps to create a communication channel between different classes while keeping alive, at the same time, the fundamental MVVM concept of “separation of concerns”. In fact, when a class sends a message, it doesn’t know who’s going to receive it: it’s the receiver class that will simply register itself to handle specific kind of messages.

To send a message in Caliburn Micro 1.5 we used the **Publish()** method of the **IEventAggregator** class, that simply dispatched the object using the UI thread. In Caliburn Micro 2.0 this method doesn’t’ exists anymore and it’s been replaced with multiple methods, that support a greater number of scenarios, like sending messages on a background thread, or using the UI thread but in an asynchronous way.

If you want to replace the **Publish()** method without changing the old behavior (so the message is sent in a synchronous way on the UI thread), you just need to use the new **PublishOnUIThread()** method.

Otherwise, if you want to improve performances and avoid to overload the UI thread, you can use the new asynchronous version, which is **PublishOnUIThreadAsync()**: however, by doing this, you’ll need a bit more work to complete the porting; since the metod is asynchronous , you’ll have to add as prefix the **await** keyword and mark the container method with the **async** one. But, most of all, you’ll have to verify that you’re correctly managing the asynchronous pattern: for instance, if your method that sends the message is marked as **void** and it’s not an event handler, you should change it so that it returns a **Task,** so that it can be correctly awaited by the caller.

<pre class="brush: csharp;">private async Task SendMessage()
{
  
    await eventAggregator.PublishOnUIThreadAsync(new SimpleMessage());
  
}
</pre>

There’s also another useful new method offered by the **IEventAggregator** class, which is called **PublishOnBackgroundThread()**: by using it, the **Handle()** method that will receive it will be executed in a background thread instead of the UI thread. It’s useful when the receiver class needs to perform many CPU consuming tasks when a message is received.

### New namespace for the Message class

In my application I found myself multiple times using the **Message.Attach** attached property, which can be useful to manage in a ViewModel events that are raised by controls in the UI. In the previous Caliburn Micro version, the **Message** class was define inside the **Caliburn.Micro** assemply. It means that, to use it, you had added the following namespace in the XAML definition:

<pre class="brush: xml;">xmlns:micro="clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro
</pre>

Now the assembly that contains the **Message** class is changed to **Caliburn.Micro.Platform**: as a consequence, you’ll have to change all your XAML namespaces definition to

<pre class="brush: xml;">xmlns:micro="clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro.Platform"
</pre>

### Managing the application bar

To manage the application bar in a Caliburn way I’ve used in my project a third party component made by Kamran Ayub, called **CaliburnBindableAppBar.** I’ve talked abut it in details <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">in the following blog post</a>. The problem comes if, like me, you’ve installed the component using NuGet: the available version, in fact, it’s compiled against Caliburn Micro 1.5 and it won’t work if you upgrade your project to Caliburn Micro 2.0. However, the developer has already updated the project: on <a href="https://github.com/kamranayub/CaliburnBindableAppBar" target="_blank">GitHub</a> you can see that the component already supports Caliburn Micro 2.0, as stated in the changelog. I’ve already contacted the developer and he promised me that he’s going to update the package on NuGet soon; however, if you need to use the updated application bar right now, the solution is simple: just download the project from GitHub and compile it by yourself or add it to your solution.

### That’s all!

If you had a similar experience and you found other changes that are worthes to be mentioned, feel free to leave a comment!