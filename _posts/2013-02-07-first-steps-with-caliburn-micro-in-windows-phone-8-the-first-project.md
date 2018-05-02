---
id: 1382
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; The first project'
date: 2013-02-07T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1382
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/
categories:
  - Windows Phone
tags:
  - Caliburn
  - Windows Phone
---
In the [previous post](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/ "First steps with Caliburn Micro in Windows Phone 8 &ndash; The theory") we’ve seen the theory behind the Caliburn Micro project and why is different from other toolkits (like MVVM Light) that are available to implement the MVVM pattern in a Windows Phone application.

In this post we’ll start to do some practice and we’ll see how to create an application with the MVVM pattern that uses the Caliburn Micro framework.

UPDATE: the section of this post about using the naming convention to link the View and the ViewModel has been updated, to address an issue I included in the previous version of the post. Views and ViewModels should be created inside a specific namespace.

### Create the first project

First you’ll have to create a new Windows Phone project (7.5 or 8 is the same for the moment, Caliburn Micro supports both) and, using NuGet, add the **Caliburn.Micro** library: NuGet will take care of satisfying all the needed dependencies.

The first step when working with Caliburn Micro is to setup the **bootstrapper**, which is the class that will take care of creating all the needed infrastructure so that all the conventions and helpers available can work correctly. To do this, simply create a new class which should inherit from the **PhoneBootstrapper** class, like in the following sample:

<pre class="brush: csharp;">public class Bootstrapper : PhoneBootstrapper
{
    PhoneContainer container;

    protected override void Configure()
    {
        container = new PhoneContainer();

        container.RegisterPhoneServices(RootFrame);
        container.PerRequest&lt;MainPageViewModel&gt;();

        AddCustomConventions();
    }

    static void AddCustomConventions()
    {
        //ellided  
    }

    protected override object GetInstance(Type service, string key)
    {
        return container.GetInstance(service, key);
    }

    protected override IEnumerable&lt;object&gt; GetAllInstances(Type service)
    {
        return container.GetAllInstances(service);
    }

    protected override void BuildUp(object instance)
    {
        container.BuildUp(instance);
    }
}</pre>

This sample is a standard bootstrapper for Caliburn Micro: unless you need to do something special, like registering some custom conventions, you can basically copy and past this code in your every application. The only thing you’ll have to change is the ViewModels registration: inside the **Configure** method you’ll have to register in the built in dependency injection container every view model and every service you’re going to use in your application. In this example, we’re going to register the class **MainPageViewModel** using the **PerRequest** method: this means that, every time the application will require a **MainPageViewModel** object, we will get a new instance of the class.

**UPDATE:** In Caliburn Micro 1.5.2 there was a breaking change in the boostrapper’s definition: previously, the **RootFrame** was passed as parameter of the **PhoneContainer** class, while now it’s required as parameter of the **RegisterPhoneServices()** method.

Now we need to initialize the bootstrapper: in Windows Phone we simply do that by adding it as as resource in the global application resources in the **App.xaml** file.

<pre class="brush: xml;">&lt;Application
    x:Class="CaliburnMicro.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
    xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
    xmlns:caliburnMicro="clr-namespace:CaliburnMicro"&gt;

    &lt;!--Application Resources--&gt;
    &lt;Application.Resources&gt;
        &lt;caliburnMicro:Bootstrapper x:Key="bootstrapper" /&gt;
    &lt;/Application.Resources&gt;

&lt;/Application&gt;</pre>

This how it looks like a standard **App.xaml** in a Caliburn Micro application. Since the bootstrapper takes care of initializing everything needed by the Windows Phone application, not just the stuff required by Caliburn Micro, we can clean also the **App.xaml.cs.** This is how it will look like in the end:

<pre class="brush: csharp;">public partial class App : Application
{
    /// &lt;summary&gt;
    /// Provides easy access to the root frame of the Phone Application.
    /// &lt;/summary&gt;
    /// &lt;returns&gt;The root frame of the Phone Application.&lt;/returns&gt;
    public static PhoneApplicationFrame RootFrame { get; private set; }

    /// &lt;summary&gt;
    /// Constructor for the Application object.
    /// &lt;/summary&gt;
    public App()
    {

        // Standard XAML initialization
        InitializeComponent();

        // Show graphics profiling information while debugging.
        if (Debugger.IsAttached)
        {
            // Display the current frame rate counters.
            Application.Current.Host.Settings.EnableFrameRateCounter = true;

            // Show the areas of the app that are being redrawn in each frame.
            //Application.Current.Host.Settings.EnableRedrawRegions = true;

            // Enable non-production analysis visualization mode,
            // which shows areas of a page that are handed off to GPU with a colored overlay.
            //Application.Current.Host.Settings.EnableCacheVisualization = true;

            // Prevent the screen from turning off while under the debugger by disabling
            // the application's idle detection.
            // Caution:- Use this under debug mode only. Application that disables user idle detection will continue to run
            // and consume battery power when the user is not using the phone.
            PhoneApplicationService.Current.UserIdleDetectionMode = IdleDetectionMode.Disabled;
        }

    }

}</pre>

We’ve removed every other initialization, since it’s not needed anymore: the bootstrapper will take care for us.

Now we are ready to start developing our application! In this first sample we’ll simply show some stuff on the screen and read the input of the user, just to familiarize with the conventions exposed by the toolkit.

### Linking the View with the ViewModel

As already anticipated in the previous post, there’s no need to manually connect the View and the ViewModel: is it enough to use a specific naming convention and Caliburn Micro will take care for you. The naming convention is the following: the ViewModel of a View will have **the same name of the View** **plus the ViewModel suffix**. Plus, you should follow also a naming convention for namespaces: the ViewModels should be created inside the **ViewModels** namespace, while the Views should be stored inside the **Views** namespace. Typically&nbsp; this goal is reached by creating two separate folders, called ViewModels and Views, in your Visual Studio project. Let’s see how it works with a real example: as for every standard Windows Phone project, you should already have a view called **MainPage.xaml**, which is the first one that is loaded when the application starts. Delete it, since we&#8217;ll need to create a new to follow the required naming convention.

Create a new folder inside your project called **Views** and create a new **MainPage.xaml** file inside (just choose one of the available templates for creating a Windows Phone page). Remember also to open the manifest file and, in the **Application UI** tab, change the **NavigationPage** field to point to the correct main page (for example, /Views/MainPage.xaml): otherwise, the application will crash at startup because it will try to launch a page that doesn&#8217;t exist.

Now let’s create a folder **ViewModels** in your project and, inside it, let’s add a new class, called **MainPageViewModel.** As you can see, the name of the class is the same of our view (**MainPage**), plus the ViewModel suffix. Now let’s do an experiment: in the **MainPageViewModel** class simply add a public constructor and, inside, show a message using a **MessageBox**.

<pre class="brush: csharp;">public class MainPageViewModel
{
    public MainPageViewModel()
    {
        MessageBox.Show("I'm the MainView!");
    }
}</pre>

Now press F5 and start the debug in Visual Studio: voil� , you’ll see the message on the emulator’s screen! Here is the magic of the naming convention in action: since the view model has the same name of the view plus the ViewModel suffix, the view model is automatically set as DataContext of the View. This way, when the view is displayed, the ViewModel is automatically initialized and the public constructor, as it happens for every class, is called.

**Important!** The naming convention’s magic make sure that view and view models are automatically connected, but it still requires you to define which are the view models available in your application, by registering them in the dependancy container in the **Configure** method of the bootstrapper, as we have seen in the beginning of the post. This means that if we’re going to add more views and view models to our application, we’ll have to remember to register them in the bootstrapper.

### Show some properties

A ViewModel without properties it’s basically useless: let’s add a property and display it in the page. The first thing to do is to start using some of the helpers that are available in Caliburn Micro: by inheriting our view model from the class **PropertyChanged** base we’ll have an easy way to support the **INotifyPropertyChanged** interface and to declare our properties to that, every time we change their value, the controls that is are binding with them are automatically refreshed to display the new value.

Here is how to define a sample property using this mechanism:

<pre class="brush: csharp;">public class MainPageViewModel: PropertyChangedBase
{
    public MainPageViewModel()
    {
        Name = "Matteo";
    }

    private string name;

    public string Name
    {
        get { return name; }
        set
        {
            name = value;
            NotifyOfPropertyChange(() =&gt; Name);
        }
    }
}</pre>

We define a property called **Name** and, in the setter, we call the **NotifyOfPropertyChange** method (provided by the toolkit) to notify the controls that something is changed. In the constructor, we simply assign a value to this property. Now we need to display it. In a standard application, we would put the **Name** property in binding with the **Text** property of a **TextBlock**, like in the following example:

<pre class="brush: xml;">&lt;TextBlock Text="{Binding Name}" /&gt;</pre>

With Caliburn Micro this is not needed because, again, we have a naming convention for binding: the name of the control (the value of the **x:Name** property) should match the name of the property in the view model. In our sample, we need simply to do this:

<pre class="brush: xml;">&lt;TextBlock x:Name="Name" /&gt;</pre>

If we launch again our application in the emulator you’ll see that the text **Matteo** will be displayed in the **TextBlock** we’ve placed in the XAML.

If you don’t like using naming conventions, you can always use the standard approach: since the ViewModel is set as DataContext of the view, as with every other MVVM based application, you can use also the standard syntax with the **Binding** markup extensions, as we’ve seen in the first sample.

### Actions, tombstoning, messaging and a lot more

There are a lot more scenarios that we haven’t faced yet. We’ll start from the next post by seeing how to manage actions and commanding using Caliburn Micro.

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
      * The first project 
          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a> 
              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a> 
                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a> 
                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a> 
                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services </a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>