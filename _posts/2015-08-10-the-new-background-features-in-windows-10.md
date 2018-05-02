---
id: 6429
title: The new background features in Windows 10
date: 2015-08-10T17:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6429
permalink: /the-new-background-features-in-windows-10/
categories:
  - Universal Apps
tags:
  - Windows
  - Windows 10
---
If you already have some experience in developing Universal Windows apps for Windows and Windows Phone 8.1, you should already be familiar with the “application lifecycle” concept. Compared to the traditional desktop model, where the application lifecycle is quite simple (basically, one app is able to run and perform tasks until it’s closed), in the mobile world things are a bit different. A mobile device like a smartphone, in fact, needs to handle many requirements which aren’t common in the desktop world, like battery usage, limited available memory, etc. 

Consequently, Universal Windows apps (both on 8.1 and 10) use a different lifecycle model, which can be summarized with the following image: 

&nbsp; 

[<img title="clip_image002" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image002" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/08/clip_image002_thumb.jpg?resize=471%2C269" width="471" height="269"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/08/clip_image002.jpg)__ 

__&nbsp; 

When an application is opened, it enters into a state called **running**: in this state, it’s able to use memory, CPU, sensors and whatever feature is available on the device. As soon as the app is **suspended** (because the user has pressed the Start button or has tapped on a notification sent by another application), after 5 seconds the process is “frozen” in memory. It means that the process will be kept in the device’s RAM memory, but it won’t be able to perform any operation. As such, every device’s resource (CPU, the camera, sensors, etc.) will be available to every other application that the user will decide to launch next. When the user decides to return to our application, the **resuming** event is triggered: the process is awaken and the application can use again all the device’s resources. 

It may happen, however, that the resources are running low: for example, the user starts to open a lot of applications or he may have suspended one or two apps which use a lot of memory (like a game). In this case, when a new application is opened, the device may not have enough memory to manage it: consequently, the operating system has the chance to **terminate** the older applications to free some memory. In this scenario, it’s the developer’s job to save the state of the application, during the suspension phase, in order to restore it in case the app is reopened after a termination. The goal of this approach is to give to the user the impression that the app has never been closed: the termination by the operating system should be completely transparent for him. 

### 

### Perform operations in background: background tasks

As you can imagine after reading how the application’s lifecycle works, Universal Windows apps don’t have the chance to perform operations when they’re suspended. The process, in fact, is “frozen” and, as such, it can’t perform any activity which involves the usage of CPU, network, etc.

To allow developers to handle the requirement to perform, somehow, some operations even when the app is suspended, the Windows Runtime has introduced the concept of **background task.** They are separate projects from the main one that contains the application, but they belong to the same solution and they access to the same app’s context (so, for example, a background task can access to the same local storage used by the application). Background tasks are usually made by a single class, which contains the code that is executed when it’s scheduled by operating system, even if the main application isn’t running.

Here is a sample definition of a background task:

<pre class="brush: csharp;">using Windows.ApplicationModel.Background; 

namespace Tasks 
{ 
    public sealed class ExampleBackgroundTask : IBackgroundTask 
    { 

        public void Run(IBackgroundTaskInstance taskInstance) 
        { 

        } 

    } 
} 

</pre>

A background task is made by a class, which implements the **IBackgroundTask** interface and manages a method called **Run()**, which is invoked when the task is activated. 

Background tasks are projects which type is Windows Runtime Component and they’re connected to the concept of **trigger**, which are events that you can subscribe to invoke a background task. There are many types of triggers in Windows Runtime: they can be used to manage periodic requirements (for example, if the task needs to run every 30 minutes), system events (for example, the user has locked / unlocked the device) or the communications with other devices (there is a set of triggers to interact with Bluetooth Low Energy devices).

I won’t go deep about background tasks in this post, since the way they work has unchanged moving from Windows 8.1 to Windows 10. You will find a wide set of new triggers, but the base concepts that drive the development are the same. If you’re interested into this subject, you can refer to the MSDN documentation which is available at <https://msdn.microsoft.com/en-us/library/windows/apps/xaml/hh977056.aspx> 

### Keep the application active when it’s suspended

The new feature added in Windows 10, which I’m going to detail in this post, is the **ExtendedExecutionSession** class, which can be used to support two new scenarios:

  1. The application needs to save some data or complete a pending operation before it’s suspended and the 5 seconds timeframe isn’t enough. 
      * The application needs to continue the execution even when it’s suspended, for example to keep track of the user’s position (a fitness app, a turn by turn navigator, etc.)</ol> 
    Let’s see how to handle both scenarios.
    
    #### 
    
    #### Complete a pending operation
    
    This scenario is managed inside the event that the **App** class provides to handle the suspension. In the standard definition of this class (which is declared inside the **App.xaml.cs** file) you’ll find the following lines of code:
    
    <pre class="brush: csharp;">private void OnSuspending(object sender, SuspendingEventArgs e) 
{ 
    var deferral = e.SuspendingOperation.GetDeferral(); 
    deferral.Complete(); 
}

</pre>
    
    By default, this code doesn’t do anything: it just asks for a deferral, which is a way to handle asynchronous operations inside the suspension event. Without using the deferral, the beginning of an asynchronous operation may cause the immediate termination of the app by the operating system, since it’s performed on a different thread than the main one. 
    
    Inside the **OnSupending()** method we can use the **ExtendedExecutionSession** class to ask for more time, so that we have the chance to complete the running operation. Here is a sample code: 
    
    <pre class="brush: csharp;">private async void OnSuspending(object sender, SuspendingEventArgs e) 
{ 
    var deferral = e.SuspendingOperation.GetDeferral(); 
    var extendedSession = new ExtendedExecutionSession(); 
    extendedSession.Reason = ExtendedExecutionReason.SavingData; 
    extendedSession.Description = "Complete synchronization"; 

    ExtendedExecutionResult result = await extendedSession.RequestExtensionAsync(); 

    if (result == ExtendedExecutionResult.Allowed) 
    { 
        UploadCompleteData(); 
    } 
    else 
    { 
        UploadBasicData(); 
    } 

    deferral.Complete(); 
} 
</pre>
    
    When we create a new **ExtendedExecutionSession** instance, we have to define: 
    
      1. The reason why we need that the running operation continues even after the suspension. In our scenario, we need to use the **SavingData** value of the **ExtendedExecutionReason** enumartor. 
          * A description that briefly summarize the operation we’re going to perform. It’s a simple string.</ol> 
        Then we can call the **RequestExtensionsAsync().** As you can imagine from the name of the method, this operation doesn’t guarantee that the request will be accepted; the operating system could decide to deny it, according to the available resources. Consequently, it’s very important to check the result of the operation, which type is **ExtendedExecutionResult.** Only if the result is **Allowed**, we are authorized to execute an operation that could take more than 5 seconds to be completed; otherwise, instead, we need to apply a workaround to satisfy the 5 seconds constraint.
        
        It’s important to remind that this approach doesn’t allow an infinite execution time: in case resources are running low, the operating system could terminate the running activity. 
        
        ### 
        
        #### Keep the application running in background
        
        The second feature provided by the **ExetendedExecutionSession** class offers a bit more flexibility: when the application is suspended, in fact, it will be kept alive in background: the process won’t be frozen, but it will be able to keep performing operations, handle events, etc. It’s like if the application in foreground, except that it isn’t visible to the user. It’s a feature which can be very useful for location tracking apps: for example, a fitness app could leverage this approach to track the user’s run even if the phone is locked and placed in the user’s pocket.
        
        This is the reason why the **ExtendedExecutionSession** class identifies this approach by assigning the **LocationTracking** value to the **Reason** property. Moreover, this feature requires that the **Location** capability is declared in the manifest file. To be honest, the operating system doesn’t force you to use the location APIs to use this feature: however, it’s important to keep in mind that the user, in the Store page, will see that your application is accessing to the user’s location, even if you aren’t actually doing it. As such, think twice before choosing to go down this path.
        
        The main difference with the previous scenario (completing a pending operation) is the position in code when the **ExtendedExceutionSession** class is used: if, in the code sample we’ve previously seen, we used it inside the **OnSuspending()** method of the **App** class, now instead we need to ask the permission to the operating system to run in background as soon as possible. Typically, this is done when the main page of the application is loaded, so the **OnNavigatedTo()** or the **Loaded**() ****events are good candidates.
        
        Here is a code sample:
        
        <pre class="brush: csharp;">protected override async void OnNavigatedTo(NavigationEventArgs e) 
{ 
    if (e.NavigationMode == NavigationMode.New) 
    { 
        var extendedSession = new ExtendedExecutionSession(); 
        extendedSession.Reason = ExtendedExecutionReason.LocationTracking; 
        extendedSession.Description = "Location tracking"; 

        ExtendedExecutionResult result = await extendedSession.RequestExtensionAsync(); 
        if (result == ExtendedExecutionResult.Allowed) 
        { 
            Debug.WriteLine("Background execution approved"); 
        } 
        else 
        { 
            Debug.WriteLine("Background execution denied"); 
        } 
    } 
} 

</pre>
        
        The first line of code makes sure that the initialization is made only when the navigation mode is **New**, which means that the page is loaded for the first time: this way, we can avoid that the initialization is repeated every time that the user, from an inner page, navigates back to the main one. If we try to initialize the **ExtendedExecutionSession** object multiple times, in fact, we’ll get an exception.
        
        The rest of the code is pretty much the same we’ve seen in the previous scenario, except that:
        
          1. We’re setting the **Reason** property of the **ExtendedExecutionSession** class to **LocationTracking**.
          2. Also in this case it’s useful to check the result of the **RequestExtensionAsync()** method, but only for logging or warning purposes. In fact, in case the app is allowed for background execution, we won’t have anything special to do: the app will simply continue to run, like if it has never been suspended. It’s important to highlight that one of the causes that may lead to a denied execution is that the **Location** capability is missing in the manifest file.
        #### It’s important to highlight that, even if the application is indeed running in background, it’s not able to interact with the UI thread: if you need to show any information to the user, you need to rely on the features provided by the platform, like notifications.
        
        The following sample code shows how to leverage notifications in a background execution scenario:
        
        <pre class="brush: csharp;">public sealed partial class MainPage : Page
{
    public MainPage()
    {
        this.InitializeComponent();
    }

    protected override async void OnNavigatedTo(NavigationEventArgs e)
    {
        if (e.NavigationMode == NavigationMode.New)
        {
            var extendedSession = new ExtendedExecutionSession();
            extendedSession.Reason = ExtendedExecutionReason.LocationTracking;
            extendedSession.Description = "Location tracking";

            ExtendedExecutionResult result = await extendedSession.RequestExtensionAsync();
            if (result == ExtendedExecutionResult.Allowed)
            {
                Debug.WriteLine("Background execution approved");
            }
            else
            {
                Debug.WriteLine("Background execution denied");
            }

            Geolocator locator = new Geolocator();
            locator.DesiredAccuracyInMeters = 0;
            locator.MovementThreshold = 500;
            locator.DesiredAccuracy = PositionAccuracy.High;
            locator.PositionChanged += Locator_PositionChanged;
        }
    }

    private void Locator_PositionChanged(Geolocator sender, PositionChangedEventArgs args)
    {
        string xml = $@"
            &lt;toast activationType='foreground' launch='args'&gt;
                &lt;visual&gt;
                    &lt;binding template='ToastGeneric'&gt;
                        &lt;text&gt;This is a toast notification&lt;/text&gt;
                        &lt;text&gt;Latitude: {args.Position.Coordinate.Point.Position.Latitude} - Longitude: {args.Position.Coordinate.Point.Position.Longitude}&lt;/text&gt;
                    &lt;/binding&gt;
                &lt;/visual&gt;
            &lt;/toast&gt;";

        XmlDocument doc = new XmlDocument();
        doc.LoadXml(xml);

        ToastNotification notification = new ToastNotification(doc);
        ToastNotifier notifier = ToastNotificationManager.CreateToastNotifier();
        notifier.Show(notification);
    }
}
</pre>
        
        After subscribing for background execution, we initialize the **Geolocator** class and we subscribe to the **PositionChanged** event, which is triggered every time the geo localization services detects a new position. When this event happens, we prepare the payload of a toast notification with the info about the detected longitude and latitude and then we display it, using the **ToastNotification** and **ToastNotificationManager** classes. We can easily test this application using the Windows 10 Mobile emulator: just open the app and, by leveraging the Location simulator in the Additional tools, just place a pushpin in a random map location. Every time you’ll perform this operation, you’ll see a notification with the coordinate of the selected place. However, since we requested background execution, the **PositionChanged** event handler will be triggered even if the app is suspended. Let’s repeat the same test (placing random pushpins on the map) but, before, suspend the app: you’ll notice that the notifications will be displayed anyway, since the app is indeed still running, even if it’s not visible.
        
        As you can see, this features allows a lot of flexibility; consequently, there are some constraints to prevent that an app could negatively affect the battery life or the performances of the devices. First, only one application at a time can leverage the background execution; if the user opens another application that uses the same feature, the already running one will be stopped and suspended. In addition, also in this case the operating system continuously monitors the available system resources: if they are running low, it will be able to kill our running app and suspend it.
        
        ### 
        
        ### Wrapping up
        
        The new background execution feature is very simple to manage but, at the same time, very powerful and it opens up scenarios that it wasn’t possible to support in a Windows or Windows Phone 8.1 application. You can find with a sample project that shows how to use the **ExtendedExecutionSession** API on my GitHub repository [https://github.com/qmatteoq/Windows10-Samples](https://github.com/qmatteoq/Windows10-Samples "https://github.com/qmatteoq/Windows10-Samples"). Happy coding!