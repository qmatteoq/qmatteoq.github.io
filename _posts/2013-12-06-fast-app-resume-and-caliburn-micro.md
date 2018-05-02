---
id: 5102
title: Fast App Resume and Caliburn Micro
date: 2013-12-06T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=5102
permalink: /fast-app-resume-and-caliburn-micro/
categories:
  - Windows Phone
tags:
  - Caliburn
  - Fast App Resume
  - Windows Phone
---
Fast App Resume is one of the new features introduced in Windows Phone 8 that allows developers to introduce a smoother experience in their applications. The standard navigation experience is that, when a user suspends the application, he’s able to resume it only by pressing the Back button or by using the task switcher. If he launches the same application using the main tile or the icon in the application list, the suspended process is terminated and a new instance of the app is executed. The result is that, whatever the user was doing, he will have to start from scratch. 

Fast App Resume introduces a more user friendly experience: the user is able to definitely quit from the application only by pressing the Back button in the main page. In every other case (including tapping the main tile or the icon in the application list) the previous instance is resumed. This way the user doesn’t have to remember that, to resume the work he was previously doing, he has to use the task switcher: regardless of the way he is going to open the app, he’ll be able to keep using the app from where he left. Anyway, Fast App Resume is a double edged sword: not always the user has the need to resume his work, but he just need to start from beginning.&nbsp; Think about, for example, a Twitter client: most of the time, when the user opens the app he wants to see his timeline and not to keep reading the tweet he was looking at the last time he used the app (that could have happened many hours before). So, my suggestion is: use Fast App Resume carefully!

After this brief introduction, let’s get back to the core of the post: implementing Fast App Resume means editing the manifest and apply some changes to the navigation events of the Windows Phone’s main frame (which class is called **PhoneApplicationFrame** and takes care of managing all the rendering of the pages and the navigation system). From a developer’s point of view, the operation needs to be done in the **App.xaml.cs** class: when the **PhoneApplicationFrame** object is created, you have to subscribe to the **Navigated** and **Navigating** events and apply some code, that we’ll see later.

The problem when you work with Caliburn Micro is that all the application’s setup is made by the boostrapper class: initialization is removed from the **App.xaml.cs** class, so you don’t have a way anymore to make the needed changes. A good alternative is to use the boostrapper class, since it replaces the **App** one: the problem is that, by default, you don’t have a way to access to the **PhoneApplicationFrame** class in the bootstrapper.

But there’s a workaround  <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/12/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />The boostrapper class, by default, initializes the **PhoneApplicationFrame** under the hood, but it offers a method that can be used to customize the process. It’s useful in case you want to override, for example, the default frame with another one (like the **TransitionFrame** included in the Windows Phone Toolkit that offers built-in support to transitions). The name of the method is **CreatePhoneApplicationFrame** and should return a **PhoneApplicationFrame** object (which is the basic class that every frame class inherits from). In our case, we can use it just to define a private instance of the **PhoneApplicationFrame** class, that we’re going to use to subscribe to the **Navigated** and **Navigating** events.

Here is the code:

<pre class="brush: csharp;">public class AppBootstrapper : PhoneBootstrapperBase
 {
     private PhoneContainer container;
     private PhoneApplicationFrame rootFrame;

     public AppBootstrapper()
     {
         Start();
     }

     protected override PhoneApplicationFrame CreatePhoneApplicationFrame()
     {
         rootFrame = new PhoneApplicationFrame();
         return rootFrame;
     }
}</pre>

We’re basically doing the same things that the boostrapper does under the hood: the difference is that, this time, since we have an explicit reference to the **PhoneApplicationFrame** object, we can use it in our code.

What we’re going to use it for? To properly support Fast App Resume we need to intercept two navigation events: **Navigated** (that is triggered every time the user has navigated to another page) and **Navigating** (which, instead, is triggered just before the navigation is performed). It’s important to keep in mind, at this point, that navigating to another page doesn’t necessarily mean navigating from an application’s page to another page. It can be also a suspension (navigation from a page to the start screen) or a restore or launch (navigation from the start screen to a page). 

The first step to continue our implementation is to change the **Configure()** method of the boostrapper and add two event handlers, to subscribe to the two events, like in the following sample: 

<pre class="brush: csharp;">protected override void Configure()
{
    container = new PhoneContainer();
    if (!Execute.InDesignMode)
        container.RegisterPhoneServices(RootFrame);

    container.PerRequest&lt;MainPageViewModel&gt;();

    AddCustomConventions();

    rootFrame.Navigated += rootFrame_Navigated;
    rootFrame.Navigating += rootFrame_Navigating;
}

void rootFrame_Navigating(object sender, NavigatingCancelEventArgs e)
{
    throw new NotImplementedException();
}

void rootFrame_Navigated(object sender, NavigationEventArgs e)
{
    throw new NotImplementedException();
}</pre>

To better understand the code we’re going to write, let’s explains how Fast App Resume works from a developer point of view: usually, when you tap on the main tile, you launch a new instance of the app with, as navigation uri, the main page of the application. The same happens when Fast App Resume is enabled: the difference is that, this time, the previous instance of the app will be resumed, with the last page visited by the user already at the top of the navigation stack. The problem is that, since the main tile triggers a navigation to the main page, the user won’t see the last opened page, but the main one. Other than being an incorrect behavior, it will cause also a navigation issue: if the user presses Back on the main page, he will be redirected to the last opened page, instead of closing the app. So, the goal for the developer that correctly wants to support Fast App Resume is to detect this scenario and to cancel the navigation towards the main page: this way, the user will correctly stay on the last opened page.

To achieve this goal we’re going to use the **Navigated** and **Navigating** events we’ve previously defined.

We need to use the **Navigated** event to understand the navigation’s type that has been triggered: this event returns a parameter called **NavigationMode,** which explains which type of navigation has been issued, like **Back**, **Forward**, **New**, etc. One of the possible values is **Reset**: this is the one we’re looking for, since it’s triggered when the user has opened the application using the main tile or the icon in the application list but Fast App Resume is enabled. We need to store this information for a later use: it’s required because the **Navigated** event is the only one that can give us this information, but, since the navigation has already been completed at this time, we can’t cancel it. So we’re going to define a global property in the boostrapper, which type is **bool**, and we’re going to set it to true in case the **NavigationMode** property is equal to **Reset.** In an application that supports Fast App Resume, we’re going to get this information when app is resumed and initial navigation to the last opened page is triggered.

<pre class="brush: csharp;">public class AppBootstrapper : PhoneBootstrapperBase
  {
      private PhoneContainer container;
      private PhoneApplicationFrame rootFrame;
      private bool reset;

      public AppBootstrapper()
      {
          Start();
      }

      protected override PhoneApplicationFrame CreatePhoneApplicationFrame()
      {
          rootFrame = new PhoneApplicationFrame();
          return rootFrame;
      }

      protected override void Configure()
      {
          container = new PhoneContainer();
          if (!Execute.InDesignMode)
              container.RegisterPhoneServices(RootFrame);

          container.PerRequest&lt;MainPageViewModel&gt;();
          container.PerRequest&lt;Page2ViewModel&gt;();

          AddCustomConventions();

          rootFrame.Navigated += rootFrame_Navigated;
          rootFrame.Navigating += rootFrame_Navigating;
      }

      void rootFrame_Navigating(object sender, NavigatingCancelEventArgs e)
      {
         throw new NotImplementedException();
      }

      void rootFrame_Navigated(object sender, NavigationEventArgs e)
      {
          reset = e.NavigationMode == NavigationMode.Reset;
      }
}</pre>

Now that we have this information, we are ready to cancel the navigation to the main page in case the user opens the app from the main tile and a previous instance of the app is already available in memory. To satisfy this requirement, we’re going to use the **Navigating** event, that we’re going to use to intercept the navigation to the main page of the app before it’s completed. This second navigation is triggered immediately after the first one to the last opened page is completed (the one we intercepted with the **Navigated** event). We need to cancel this second navigation if the following conditions are satisfied:

  * Fast Application Resume is enabled. 
      * The **NavigationMode** parameter that we got in the **Navigated** event is equal to **Reset**. 
          * The Uri of the page where the user is navigating to is the main page. </ul> 
        Here is how these conditions are translated into code:
        
        <pre class="brush: csharp;">void rootFrame_Navigating(object sender, NavigatingCancelEventArgs e)
{
    if (reset && e.IsCancelable && e.Uri.OriginalString == "/Views/MainPage.xaml")
    {
        e.Cancel = true;
        reset = false;
    }
}</pre>
        
        We check the value of the boolean property we’ve defined before (called **reset**) and we check the navigation uri (stored in the **NavigatingCancelEventArgs** parameter). Of course, you’ll have to adapt your code according to the position and the name of the main page of your application: usually, it’s the same that is set in the manifest file, in the field called **Navigating page**. If these conditions are satisfied, we’re going to set the **Cancel** property of the method’s parameter to **true**: this way, the navigation to the main page will be canceled and the user will stay on the last visited page.
        
        Here is the full code of the bootsrapper to properly support Fast App Resume:
        
        <pre class="brush: csharp;">public class AppBootstrapper : PhoneBootstrapperBase
{
    private PhoneContainer container;
    private PhoneApplicationFrame rootFrame;
    private bool reset;

    public AppBootstrapper()
    {
        Start();
    }

    protected override PhoneApplicationFrame CreatePhoneApplicationFrame()
    {
        rootFrame = new PhoneApplicationFrame();
        return rootFrame;
    }

    protected override void Configure()
    {
        container = new PhoneContainer();
        if (!Execute.InDesignMode)
            container.RegisterPhoneServices(RootFrame);

        container.PerRequest&lt;MainPageViewModel&gt;();
        container.PerRequest&lt;Page2ViewModel&gt;();

        AddCustomConventions();

        rootFrame.Navigated += rootFrame_Navigated;
        rootFrame.Navigating += rootFrame_Navigating;
    }

    void rootFrame_Navigating(object sender, NavigatingCancelEventArgs e)
    {
        if (reset && e.IsCancelable && e.Uri.OriginalString == "/Views/MainPage.xaml")
        {
            e.Cancel = true;
            reset = false;
        }
    }

    void rootFrame_Navigated(object sender, NavigationEventArgs e)
    {
        reset = e.NavigationMode == NavigationMode.Reset;
    }

    protected override object GetInstance(Type service, string key)
    {
        var instance = container.GetInstance(service, key);
        if (instance != null)
            return instance;

        throw new InvalidOperationException("Could not locate any instances.");
    }

    protected override IEnumerable&lt;object&gt; GetAllInstances(Type service)
    {
        return container.GetAllInstances(service);
    }

    protected override void BuildUp(object instance)
    {
        container.BuildUp(instance);
    }

    private static void AddCustomConventions()
    {
        ConventionManager.AddElementConvention&lt;Pivot&gt;(Pivot.ItemsSourceProperty, "SelectedItem", "SelectionChanged")
            .ApplyBinding =
            (viewModelType, path, property, element, convention) =&gt;
            {
                if (ConventionManager
                    .GetElementConvention(typeof (ItemsControl))
                    .ApplyBinding(viewModelType, path, property, element, convention))
                {
                    ConventionManager
                        .ConfigureSelectedItem(element, Pivot.SelectedItemProperty, viewModelType, path);
                    ConventionManager
                        .ApplyHeaderTemplate(element, Pivot.HeaderTemplateProperty, null, viewModelType);
                    return true;
                }

                return false;
            };

        ConventionManager.AddElementConvention&lt;Panorama&gt;(Panorama.ItemsSourceProperty, "SelectedItem",
            "SelectionChanged").ApplyBinding =
            (viewModelType, path, property, element, convention) =&gt;
            {
                if (ConventionManager
                    .GetElementConvention(typeof (ItemsControl))
                    .ApplyBinding(viewModelType, path, property, element, convention))
                {
                    ConventionManager
                        .ConfigureSelectedItem(element, Panorama.SelectedItemProperty, viewModelType, path);
                    ConventionManager
                        .ApplyHeaderTemplate(element, Panorama.HeaderTemplateProperty, null, viewModelType);
                    return true;
                }

                return false;
            };
    }
}</pre>
        
        Before testing our work, there’s one important thing to do: enable Fast App Resume in the manifest. Unfortunately, this option isn’t supported by the visual editor, but you’ll have to manually edit the XML file: right click on the **WMAppManifest.xml** file in the **Properties** folder of your project and choose **View code**. You’ll find the following section:
        
        <pre class="brush: csharp;">&lt;Tasks&gt;
  &lt;DefaultTask Name="_default" NavigationPage="Views/MainPage.xaml" /&gt;
&lt;/Tasks&gt;</pre>
        
        To enable Fast App Resume you’ll have to add a new attribute to the **DefaultTask** node called **ActivationPolicy** and set it to **Resume**, like in the following sample:
        
        <pre class="brush: csharp;">&lt;Tasks&gt;
  &lt;DefaultTask Name="_default" NavigationPage="Views/MainPage.xaml" ActivationPolicy="Resume" /&gt;
&lt;/Tasks&gt;
</pre>
        
        That’s all! If you want to do some experiments with Fast App Resume, you can use the sample attached project, which simply contains two pages: the first page contains a button to navigate to the second one, which instead is empty. This project has enabled Fast App Resume, so you’ll notice that, if you navigate to the second page and you suspend the app by pressing the Start button, then you open it again using the main tile or the icon in the application list, you’ll notice that the previous instance of the app will be correctly resumed and you’ll land to the second page, instead of the main one.
        
        Happy coding!
        
        <div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:d525092e-8b71-412e-895d-f28bda6b0c8e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
          <p>
            <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/12/Caliburn.FastAppResume.zip" target="_blank">Download the sample project</a>
          </p>
        </div>