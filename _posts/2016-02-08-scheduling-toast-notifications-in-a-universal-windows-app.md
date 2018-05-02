---
id: 6525
title: Scheduling toast notifications in a Universal Windows app
date: 2016-02-08T16:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6525
permalink: /scheduling-toast-notifications-in-a-universal-windows-app/
categories:
  - Universal Apps
  - UWP
  - wpdev
tags:
  - Universal Windows Platform
  - Windows 10
---
Toast notifications are, without any doubt, one of the most used techniques when it comes to notify something to the user, even when the app isn’t running. It’s almost impossible not to miss a toast notification: it plays a sound, it’s displayed on the screen, it’s stored in the Action Center and, on the phone, it makes also the device vibrate.

A toast notification is mapped with a XML file, which describes its content and its behavior. The flexibility in defining a toast notification has been vastly improved in Windows 10, since the Universal Windows Platform has added:

  1. More ways to customize the look & feel of the notification. You can add images, multiple lines of text, etc. 
      * Support to interactive notifications. You can add interactive elements (like buttons or text boxes), which can be handled by a background task.</ol> 
    You can learn more about the new features introduced in the Universal Windows Platform regarding toast notification in the following blog post from the team: [http://blogs.msdn.com/b/tiles\_and\_toasts/archive/2015/07/02/adaptive-and-interactive-toast-notifications-for-windows-10.aspx](http://blogs.msdn.com/b/tiles_and_toasts/archive/2015/07/02/adaptive-and-interactive-toast-notifications-for-windows-10.aspx "http://blogs.msdn.com/b/tiles_and_toasts/archive/2015/07/02/adaptive-and-interactive-toast-notifications-for-windows-10.aspx")
    
    In this post I would like to focus on the ways you can send a toast notification, specifically on scheduled toasts, since you may find some challenges in implementing them in the proper way.
    
    ### Sending toast notifications
    
    There are multiple ways to send a toast notification to the user:
    
      1. **Within the app:** the Universal Windows Platform includes APIs like **ToastNotification** and **ToastNotificationManager** which can be used to send a toast notification when the app is running in foreground. 
          * **From a background task**: the same APIs can be used also in a background task, so that toast notifications can be sent also when the app isn’t running. 
              * **Push notifications**: a toast can be sent by a backend and received also when the app isn’t running. In this case, the app subscribes to a service offered by Microsoft (called WNS) and receives back a Url, which identifies the unique channel for that device. When the backend wants to send a push notification to that device, it executes a HTTP POST request to the Url including, in the body, the XML that describes the notification. 
                  * **Scheduled notifications**: by using the same APIs you use within the app you can create a toast notification and schedule it to be displayed at a specific time and date, even if the application isn’t running. This is the scenario we’re going to focus from now on.</ol> 
                ### Creating a scheduled toast notification
                
                Creating a scheduled toast notification is easy and you leverage the same APIs you would use for a standard toast notification sent by the app or by a background task. Here is a sample code:
                
                <pre class="brush: csharp;">private void OnScheduleToast(object sender, RoutedEventArgs e)
{
    string xml = @"&lt;toast&gt;
            &lt;visual&gt;
            &lt;binding template=""ToastGeneric""&gt;
                &lt;text&gt;Hello!&lt;/text&gt;
                &lt;text&gt;This is a scheduled toast!&lt;/text&gt;
            &lt;/binding&gt;
            &lt;/visual&gt;
        &lt;/toast&gt;";

    XmlDocument doc = new XmlDocument();
    doc.LoadXml(xml);

    ScheduledToastNotification toast = new ScheduledToastNotification(doc, DateTimeOffset.Now.AddSeconds(10));
    ToastNotificationManager.CreateToastNotifier().AddToSchedule(toast);
}
</pre>
                
                The first step is to define the XML with the toast content. To learn how to define a toast, <a href="http://blogs.msdn.com/b/tiles_and_toasts/archive/2015/07/02/adaptive-and-interactive-toast-notifications-for-windows-10.aspx" target="_blank">you can refer to the documentation</a> and you can get some help using <a href="https://www.microsoft.com/en-us/store/apps/notifications-visualizer/9nblggh5xsl1" target="_blank">Notifications Visualizer</a>, a Windows Store app by Microsoft that is able to give you an instant preview of how a specific XML will be rendered. 
                
                [<img title="snip_20160205175523" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="snip_20160205175523" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2016/02/snip_20160205175523_thumb.png?resize=486%2C243" width="486" height="243"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2016/02/snip_20160205175523.png)
                
                Once you have the XML, you need to use it to create a **XmlDocument** object by calling the **LoadXml()** method and passing, as parameter, the XML string. Be aware that there are multiple classes called **XmlDocument** in the Universal Windows Platform: the one required by your scenario belongs to the **Windows.Data.Xml.Dom** namespace.
                
                The last step is to create a new **ScheduledToastNotification** object, which is very similar to the basic **ToastNotification** one. The difference is that, this time, other than the **XmlDocument** object with the toast definition, you have to specificy also the date and time when the notification will be displayed, using a **DateTimeOffset** object. In the sample, we’re scheduling the notification to be display after 10 seconds that this code is executed. In the end, we schedule the notification by calling the **AddToSchedule()** method of the **ToastNotifier** object, which you can get by calling the **CreateToastNotifier()** method of the **ToastNotificationManager** class.
                
                If you don’t like working with XML, you can use the Notifications Extensions library available on <a href="https://www.nuget.org/packages/NotificationsExtensions.Win10/" target="_blank">NuGet</a> and documented at [http://blogs.msdn.com/b/tiles\_and\_toasts/archive/2015/08/20/introducing-notificationsextensions-for-windows-10.aspx](http://blogs.msdn.com/b/tiles_and_toasts/archive/2015/08/20/introducing-notificationsextensions-for-windows-10.aspx "http://blogs.msdn.com/b/tiles_and_toasts/archive/2015/08/20/introducing-notificationsextensions-for-windows-10.aspx") This library gives you a set of classes and methods to create notifications and, under the hood, it takes care of generating the proper XML for you. For example, here is how the previous code looks like using the Notifications Extensions library:
                
                <pre class="brush: csharp;">private void OnScheduleToast(object sender, RoutedEventArgs e)
{
    ToastContent toastContent = new ToastContent
    {
        Visual = new ToastVisual
        {
            TitleText = new ToastText
            {
                Text = "Hello!"
            },
            BodyTextLine1 = new ToastText
            {
                Text = "This is a scheduled toast!"
            }
        }
    };

    XmlDocument doc = toastContent.GetXml();

    ScheduledToastNotification toast = new ScheduledToastNotification(doc, DateTimeOffset.Now.AddSeconds(10));
    ToastNotificationManager.CreateToastNotifier().AddToSchedule(toast);
}
</pre>
                
                ### Scheduled notifications and locked devices
                
                If you try the previous code on a phone and, before the notification is displayed, you lock it by pressing the power button, you’ll realize that the notification won’t be displayed. As soon as you press the power button again to unlock the phone, you’ll see the notification appearing. What’s happening? The reason is that, by default, apps aren’t allowed to interact with the device when they’re locked, but they require a permission to do that. It’s a typical scenario when you work with background tasks: when you register a new task using the **BackgroundTaskBuilder** class you define a code similar to the following one:
                
                <pre class="brush: csharp;">protected override async void OnNavigatedTo(NavigationEventArgs e)
{
    if (BackgroundTaskRegistration.AllTasks.All(x =&gt; x.Value.Name != "ToastTask"))
    {
        BackgroundTaskBuilder builder = new BackgroundTaskBuilder();
        builder.Name = "ToastTask";
        builder.TaskEntryPoint = "ToastsTask.CheckAnswerTask";
        builder.SetTrigger(new ToastNotificationActionTrigger());
        var status = await BackgroundExecutionManager.RequestAccessAsync();
        if (status != BackgroundAccessStatus.Denied)
        {
            builder.Register();
        }
    }
    
}
</pre>
                
                You can notice that, after defining all the properties of the task (like the name, the entry point and the trigger we want to use) we call the **RequestAccessAsync()** method of the **BackgroundExecutionManager** class: only if the request isn’t denied, we move on to perform the real registration. The **RequestAccessAsync()** method makes sure that:
                
                  1. We don’t have too many background tasks registered. On low memory devices, in fact, there’s a maximum number of tasks that can be registered and, if it has been reached, the OS will deny the request.
                  2. The background task is granted access to interact with the device also when it’s locked.
                
                As you can see, the second scenario is the one we need also for our scheduled toast notification: without this approval from the OS, we won’t be able to wake up the phone even if it’s locked. However, there’s a catch: the fact that we’re using scheduled toast notification doesn’t mean that we are necessarly using also a background task in our application. The problem is that, if we try to call the **BackgroundExecutionManager.RequestAccessAsync()** method without having a background task registered, we’ll get an exception.
                
                The workaround is simple: register a fake background task. We don’t even need to add a Windows Runtime Component to our project: we just need to declare, in the manifest file, a fake background task. Open the manifest file of your app, go into the **Declarations** section and, from the dropdown menu, adds the **Background task** item. Then choose:
                
                  1. As task type, **System**.
                  2. As entry point, any value (for example, **Test**). It doesn’t have to be a real entry point for a task, since we won’t try to register the task for real.
                
                [<img title="snip_20160205175826" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="snip_20160205175826" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2016/02/snip_20160205175826_thumb.png?resize=359%2C308" width="359" height="308"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2016/02/snip_20160205175826.png)
                
                That’s all: the declaration in the manifet is enough for our scenario, since it will allow the **BackgroundExecutionManager.RequestAccessAsync()** method to be execute without any error. Now we just have to call this method when the app is starting, for example in the **OnNavigatedTo()** method of the page:
                
                <pre class="brush: csharp;">protected override async void OnNavigatedTo(NavigationEventArgs e)
{
    await BackgroundExecutionManager.RequestAccessAsync();
}
</pre>
                
                That’s all. Now if you repeat the test of scheduling a notification and locking the phone before it’s displayed, you’ll correctly see the phone waking up and displaying the toast, like if it happens for regular push notifications.
                
                Happy coding!